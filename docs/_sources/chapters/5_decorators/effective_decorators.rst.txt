2. Effective decorators: avoid common mistakes
**********************************************

While decorators are a great feature of Python, they are not exempt from issues if used
incorrectly. In this section, we will see some common issues to avoid in order to create
effective decorators.

2.1. Preserving data about the original wrapped object
++++++++++++++++++++++++++++++++++++++++++++++++++++++

One of the most common problems when applying a decorator to a function is that some of
the properties or attributes of the original function are not maintained, leading to
undesired, and hard-to-track, side-effects.

To illustrate this we show a decorator that is in charge of logging when the function is
about to run:

.. code-block:: python

    def trace_decorator(function):
        def wrapped(*args, **kwargs):
            logger.info("running %s", function.__qualname__)
            return function(*args, **kwargs)
    return wrapped

Now, let's imagine we have a function with this decorator applied to it. We might initially
think that nothing of that function is modified with respect to its original definition:

.. code-block:: python

    @trace_decorator
    def process_account(account_id):
        """Process an account by Id."""
        logger.info("processing account %s", account_id)
        ...

But maybe there are changes.

The decorator is not supposed to alter anything from the original function, but, as it turns
out since it contains a flaw it's actually modifying its name and docstring, among other
properties.

Let's try to get help for this function:

.. code-block:: python

    >>> help(process_account)
    Help on function wrapped in module decorator_wraps_1:
    wrapped(*args, **kwargs)

And let's check how it's called:
.. code-block:: python

    >>> process_account.__qualname__
    'trace_decorator.<locals>.wrapped'

We can see that, since the decorator is actually changing the original function for a new one
(called ``wrapped``), what we actually see are the properties of this function instead of those
from the original function.

If we apply a decorator like this one to multiple functions, all with different names, they
will all end up being called wrapped, which is a major concern (for example, if we want to
log or trace the function, this will make debugging even harder).

Another problem is that, in case we placed docstrings with tests on these functions, they
will be overridden by those of the decorator. As a result, the docstrings with the test we
want will not run when we call our code with the ``doctest`` module.

The fix is simple, though. We just have to apply the wraps decorator in the internal
function (``wrapped``), telling it that it is actually wrapping function :

.. code-block:: python

    def trace_decorator(function):
        @wraps(function)
        def wrapped(*args, **kwargs):
            logger.info("running %s", function.__qualname__)
            return function(*args, **kwargs)

        return wrapped

Now, if we check the properties, we will obtain what we expected in the first place.
Check help for the function, like so:

.. code-block:: python

    >>> Help on function process_account in module decorator_wraps_2:
    process_account(account_id)
    Process an account by Id.

And verify that its qualified name is correct, like so:

.. code-block:: python

    >>> process_account.__qualname__
    'process_account'

Most importantly, we recovered the unit tests we might have had on the docstrings! By
using the wraps decorator, we can also access the original, unmodified function under the
``__wrapped__`` attribute. Although it should not be used in production, it might come in
handy in some unit tests when we want to check the unmodified version of the function.

In general, for simple decorators, the way we would use ``functools.wraps`` would
typically follow the general formula or structure:

.. code-block:: python

    def decorator(original_function):
        @wraps(original_function)
        def decorated_function(*args, **kwargs):
            # modifications done by the decorator ...
            return original_function(*args, **kwargs)

    return decorated_function

.. note:: Always use ``functools.wraps`` applied over the wrapped function, when creating a decorator, as shown in the preceding formula.

2.2. Dealing with side-effects in decorators
++++++++++++++++++++++++++++++++++++++++++++

In this section, we will learn that it is advisable to avoid side-effects in the body of the
decorator. There are cases where this might be acceptable, but the bottom line is that, if in
case of doubt, decide against it, for the reasons that are explained ahead. Everything that
the decorator needs to do aside from the function that it's decorating should be placed in
the innermost function definition, or there will be problems when it comes to importing.

Nonetheless, sometimes these side-effects are required (or even desired) to run at import
time, and the obverse applies.

We will see examples of both, and where each one applies. If in doubt, err on the side of
caution, and delay all side-effects until the very latest, right after the ``wrapped`` function is
going to be called.

Next, we will see when it's not a good idea to place extra logic outside the ``wrapped``
function.

2.2.1. Incorrect handling of side-effects in a decorator
--------------------------------------------------------

Let's imagine the case of a decorator that was created with the goal of logging when a
function started running and then logging its running time:

.. code-block:: python

    def traced_function_wrong(function):
        logger.info("started execution of %s", function)
        start_time = time.time()

        @functools.wraps(function)
        def wrapped(*args, **kwargs):
            result = function(*args, **kwargs)
            logger.info(
                "function %s took %.2fs",
                function,
                time.time() - start_time
            )
            return result
        return wrapped

Now we will apply the decorator to a regular function, thinking that it will work just fine:

.. code-block:: python

    @traced_function_wrong
    def process_with_delay(callback, delay=0):
        time.sleep(delay)
        return callback()

This decorator has a subtle, yet critical bug in it.
First, let's import the function, call it several times, and see what happens:

.. code-block:: python

    >>> from decorator_side_effects_1 import process_with_delay
    INFO:started execution of <function process_with_delay at 0x...>

Just by importing the function, we will notice that something's amiss. The logging line
should not be there, because the function was not invoked.

Now, what happens if we run the function, and see how long it takes to run? Actually, we
would expect that calling the same function multiple times will give similar results:

.. code-block:: python

    >>> main()
    ...
    INFO:function <function process_with_delay at 0x> took 8.67s
    >>> main()
    ...
    INFO:function <function process_with_delay at 0x> took 13.39s
    >>> main()
    ...
    INFO:function <function process_with_delay at 0x> took 17.01s

Every time we run the same function, it takes longer! At this point, you have probably
already noticed the (now obvious) error.

Remember the syntax for decorators. ``@traced_function_wrong`` actually means the
following: ``process_with_delay = traced_function_wrong(process_with_delay)``. And this will run when the
module is imported. Therefore, the time that is set in the
function will be the one at the time the module was imported. Successive calls will compute
the time difference from the running time until that original starting time. It will also log at
the wrong moment, and not when the function is actually called.

Luckily, the fix is also very simple: we just have to move the code inside the wrapped
function in order to delay its execution:

.. code-block:: python

    def traced_function(function):
        @functools.wraps(function)
        def wrapped(*args, **kwargs):
            logger.info("started execution of %s", function.__qualname__)
            start_time = time.time()
            result = function(*args, **kwargs)
            logger.info(
                "function %s took %.2fs",
                function.__qualname__,
                time.time() - start_time
            )
            return result
        return wrapped

With this new version, the previous problems are resolved.

If the actions of the decorator had been different, the results could have been much more
disastrous. For instance, if it requires that you log events and send them to an external
service, it will certainly fail unless the configuration has been run right before this has been
imported, which we cannot guarantee. Even if we could, it would be bad practice. The
same applies if the decorator has any other sort of side-effect, such as reading from a file,
parsing a configuration, and many more.

2.2.2. Requiring decorators with side-effects
---------------------------------------------

Sometimes, side-effects on decorators are necessary, and we should not delay their
execution until the very last possible time, because that's part of the mechanism which is
required for them to work.

One common scenario for when we don't want to delay the side-effect of decorators is
when we need to register objects to a public registry that will be available in the module.

For instance, going back to our previous event system example, we now want to only
make some events available in the module, but not all of them. In the hierarchy of events,
we might want to have some intermediate classes that are not actual events we want to
process on the system, but some of their derivative classes instead.

Instead of flagging each class based on whether it's going to be processed or not, we could
explicitly register each class through a decorator.

In this case, we have a class for all events that relate to the activities of a user. However, this
is just an intermediate table for the types of event we actually want, namely
``UserLoginEvent`` and ``UserLogoutEvent``:

.. code-block:: python

    EVENTS_REGISTRY = {}

    def register_event(event_cls):
        """Place the class for the event into the registry to make it
        accessible in
        the module.
        """
        EVENTS_REGISTRY[event_cls.__name__] = event_cls
        return event_cls

    class Event:
        """A base event object"""

    class UserEvent:
        TYPE = "user"

    @register_event
    class UserLoginEvent(UserEvent):
        """Represents the event of a user when it has just accessed the
        system."""

    @register_event
    class UserLogoutEvent(UserEvent):
        """Event triggered right after a user abandoned the system."""

When we look at the preceding code, it seems that ``EVENTS_REGISTRY`` is empty, but after
importing something from this module, it will get populated with all of the classes that are
under the register_event decorator:

.. code-block:: python

    >>> from decorator_side_effects_2 import EVENTS_REGISTRY
    >>> EVENTS_REGISTRY
    {'UserLoginEvent': decorator_side_effects_2.UserLoginEvent,
    'UserLogoutEvent': decorator_side_effects_2.UserLogoutEvent}

This might seem like it's hard to read, or even misleading, because ``EVENTS_REGISTRY`` will
have its final value at runtime, right after the module was imported, and we cannot easily
predict its value by just looking at the code.

While that is true, in some cases this pattern is justified. In fact, many web frameworks or
well-known libraries use this to work and expose objects or make them available.

It is also true that in this case, the decorator is not changing the wrapped object, nor altering
the way it works in any way. However, the important note here is that, if we were to do
some modifications and define an internal function that modifies the wrapped object, we
would still probably want the code that registers the resulting object outside it.

Notice the use of the word outside. It does not necessarily mean before, it's just not part of
the same closure; but it's in the outer scope, so it's not delayed until runtime.

2.3. Creating decorators that will always work
++++++++++++++++++++++++++++++++++++++++++++++

There are several different scenarios to which decorators might apply. It can also be the
case that we need to use the same decorator for objects that fall into these different multiple
scenarios, for instance, if we want to reuse our decorator and apply it to a function, a class,
a method, or a static method.

If we create the decorator, just thinking about supporting only the first type of object we
want to decorate, we might notice that the same decorator does not work equally well on a
different type of object. The typical example is where we create a decorator to be used on a
function, and then we want to apply it to a method of a class, only to realize that it does not
work. A similar scenario might occur if we designed our decorator for a method, and then
we want it to also apply for static methods or class methods.

When designing decorators, we typically think about reusing code, so we will want to use
that decorator for functions and methods as well.

Defining our decorators with the signature ``*args`` , and ``**kwargs`` , will make them work in
all cases, because it's the most generic kind of signature that we can have. However,
sometimes we might want not to use this, and instead define the decorator wrapping
function according to the signature of the original function, mainly because of two reasons:

- It will be more readable since it resembles the original function.
- It actually needs to do something with the arguments, so receiving ``*args`` and ``**kwargs`` wouldn't be convenient.

Consider the case on which we have many functions in our code base that require a
particular object to be created from a parameter. For instance, we pass a string, and
initialize a driver object with it, repeatedly. Then we think we can remove the duplication
by using a decorator that will take care of converting this parameter accordingly.

In the next example, we pretend that ``DBDriver`` is an object that knows how to connect and
run operations on a database, but it needs a connection string. The methods we have in our
code, are designed to receive a string with the information of the database and require to
create an instance of ``DBDriver`` always. The idea of the decorator is that it's going to take
place of this conversion automatically: the function will continue to receive a string, but
the decorator will create a ``DBDriver`` and pass it to the function, so internally we can
assume that we receive the object we need directly.

An example of using this in a function is shown in the next listing:

.. code-block:: python

    import logging
    from functools import wraps

    logger = logging.getLogger(__name__)

    class DBDriver:
        def __init__(self, dbstring):
            self.dbstring = dbstring

        def execute(self, query):
            return f"query {query} at {self.dbstring}"

    def inject_db_driver(function):
        """This decorator converts the parameter by creating a ``DBDriver``
        instance from the database dsn string.
        """

        @wraps(function)
        def wrapped(dbstring):
            return function(DBDriver(dbstring))

        return wrapped

    @inject_db_driver
    def run_query(driver):
        return driver.execute("test_function")

It's easy to verify that if we pass a string to the function, we get the result done by an
instance of ``DBDriver``, so the decorator works as expected:

.. code-block:: python

    >>> run_query("test_OK")
    'query test_function at test_OK'


But now, we want to reuse this same decorator in a class method, where we find the same
problem:

.. code-block:: python

    class DataHandler:
        @inject_db_driver
        def run_query(self, driver):
            return driver.execute(self.__class__.__name__)

We try to use this decorator, only to realize that it doesn't work:

.. code-block:: python

    >>> DataHandler().run_query("test_fails")
    Traceback (most recent call last):
    ...
    TypeError: wrapped() takes 1 positional argument but 2 were given

What is the problem? The method in the class is defined with an extra argument: ``self``. Methods are just a
particular kind of function that receives self (the object they're defined upon) as the first parameter.

Therefore, in this case, the decorator (designed to work with only one parameter, named
``dbstring``), will interpret that self is said parameter, and call the method passing the
string in the place of self, and nothing in the place for the second parameter, namely the
string we are passing.

To fix this issue, we need to create a decorator that will work equally for methods and
functions, and we do so by defining this as a decorator object, that also implements the
protocol descriptor.

The solution is to implement the decorator as a class object and make this object a
description, by implementing the ``__get__`` method.

.. code-block:: python

    from functools import wraps
    from types import MethodType

    class inject_db_driver:
        """Convert a string to a DBDriver instance and pass this to the
        wrapped function."""
        def __init__(self, function):
            self.function = function
            wraps(self.function)(self)

        def __call__(self, dbstring):
            return self.function(DBDriver(dbstring))

        def __get__(self, instance, owner):
            if instance is None:
                return self

            return self.__class__(MethodType(self.function, instance))

For now, we can say that what this decorator does is
actually rebinding the callable it's decorating to a method, meaning that it will bind the
function to the object, and then recreate the decorator with this new callable.

For functions, it still works, because it won't call the ``__get__`` method at all.
