1. What are decorators?
***********************

1.1. What are decorators in Python?
+++++++++++++++++++++++++++++++++++

Decorators were introduced in Python a long time ago as a mechanism to
simplify the way functions and methods are defined when they have to be modified after
their original definition.

One of the original motivations for this was because functions such as ``classmethod`` and
``staticmethod`` were used to transform the original definition of the method, but they
required an extra line, modifying the original definition of the function.

More generally speaking, every time we had to apply a transformation to a function, we
had to call it with the modifier function, and then reassign it to the same name the
function was originally defined with.

For instance, if we have a function called original , and then we have a function that
changes the behavior of original on top of it, called modifier , we have to write
something like the following:

.. code-block:: python

    def original(...):
    ...

    original = modifier(original)

Notice how we change the function and reassign it to the same name. This is confusing,
error-prone (imagine that someone forgets to reassign the function, or does reassign that
but not in the line immediately after the function definition, but much farther away), and
cumbersome. For this reason, some syntax support was added to the language.

The previous example could be rewritten like so:

.. code-block:: python

    @modifier
    def original(...):
    ...

This means that decorators are just syntax sugar for calling whatever is after the decorator
as a first parameter of the decorator itself, and the result would be whatever the decorator
returns.

In line with the Python terminology, and our example, modifier is what we call
the decorator, and original is the decorated function, often also called a **wrapped object**.

While the functionality was originally thought for methods and functions, the actual syntax
allows any kind of object to be decorated, so we are going to explore decorators applied to
functions, methods, generators, and classes.

One final note is that, while the name of a decorator is correct (after all, the decorator is in
fact, making changes, extending, or working on top of the wrapped function), it is not to be
confused with the decorator design pattern.

1.2. Decorate functions
+++++++++++++++++++++++

Functions are probably the simplest representation of a Python object that can be decorated.
We can use decorators on functions to apply all sorts of logic to them—we can validate
parameters, check preconditions, change the behavior entirely, modify its signature, cache
results (create a memorized version of the original function), and more.

As an example, we will create a basic decorator that implements a retry mechanism,
controlling a particular domain-level exception and retrying a certain number of times:

.. code-block:: python

    class ControlledException(Exception):
        """A generic exception on the program's domain."""

    def retry(operation):

        @wraps(operation)
        def wrapped(*args, **kwargs):
            last_raised = None
            RETRIES_LIMIT = 3

            for _ in range(RETRIES_LIMIT):
                try:
                    return operation(*args, **kwargs)
                except ControlledException as e:
                    logger.info("retrying %s", operation.__qualname__)
                    last_raised = e
            raise last_raised

        return wrapped

The use of ``@wraps`` can be ignored for now. The use of ``_`` in the for loop, means that
the number is assigned to a variable we are not interested in at the moment, because it's not
used inside the for loop (it's a common idiom in Python to name ``_`` values that are ignored).

The ``retry`` decorator doesn't take any parameters, so it can be easily applied to any
function, as follows:

.. code-block:: python

    @retry
    def run_operation(task):
        """Run a particular task, simulating some failures on its execution."""
        return task.run()

As explained at the beginning, the definition of ``@retry`` on top of ``run_operation`` is just
syntactic sugar that Python provides to actually execute ``run_operation = retry(run_operation)``.

In this limited example, we can see how decorators can be used to create a generic retry
operation that, under certain conditions (in this case, represented as exceptions that could
be related to timeouts, for example), will allow calling the decorated code multiple times.

1.2. Decorate classes
+++++++++++++++++++++

Classes can also be decorated with the same as can be applied to syntax
functions. The only difference is that when writing the code for this decorator, we have to
take into consideration that we are receiving a class, not a function.

Some practitioners might argue that decorating a class is something rather convoluted and
that such a scenario might jeopardize readability because we would be declaring some
attributes and methods in the class, but behind the scenes, the decorator might be applying
changes that would render a completely different class.

This assessment is true, but only if this technique is heavily abused. Objectively, this is no
different from decorating functions; after all, classes are just another type of object in the
Python ecosystem, as functions are. For now, we'll
explore the benefits of decorators that apply particularly to classes:

- All the benefits of reusing code and the DRY principle. A valid case of a class decorator would be to enforce that multiple classes conform to a certain interface or criteria (by making this checks only once in the decorator that is going to be applied to those many classes).
- We could create smaller or simpler classes that will be enhanced later on by decorators
- The transformation logic we need to apply to a certain class will be much easier to maintain if we use a decorator, as opposed to more complicated (and often rightfully discouraged) approaches such as metaclasses

Among all possible applications of decorators, we will explore a simple example to give an
idea of the sorts of things they can be useful for. Keep in mind that this is not the only
application type for class decorators, but also that the code we show you could have many
other multiple solutions as well, all with their pros and cons, but we chose decorators with
the purpose of illustrating their usefulness.

Recalling our event systems for the monitoring platform, we now need to transform the
data for each event and send it to an external system. However, each type of event might
have its own particularities when selecting how to send its data.

In particular, the ``event`` for a login might contain sensitive information such as credentials
that we want to hide. Other fields such as ``timestamp`` might also require some
transformations since we want to show them in a particular format. A first attempt at
complying with these requirements would be as simple as having a class that maps to each
particular ``event`` and knows how to serialize it:

.. code-block:: python

    class LoginEventSerializer:
        def __init__(self, event):
            self.event = event

        def serialize(self) -> dict:
            return {
                "username": self.event.username,
                "password": "**redacted**",
                "ip": self.event.ip,
                "timestamp": self.event.timestamp.strftime("%Y-%m-%d %H:%M")
            }

    class LoginEvent:
        SERIALIZER = LoginEventSerializer

        def __init__(self, username, password, ip, timestamp):
            self.username = username
            self.password = password
            self.ip = ip
            self.timestamp = timestamp

        def serialize(self) -> dict:
            return self.SERIALIZER(self).serialize()

Here, we declare a class that is going to map directly with the login event, containing the
logic for it: hide the password field, and format the timestamp as required.

While this works and might look like a good option to start with, as time passes and we
want to extend our system, we will find some issues:

- **Too many classes**: As the number of events grows, the number of serialization classes will grow in the same order of magnitude, because they are mapped one to one.
- **The solution is not flexible enough**: If we need to reuse parts of the components (for example, we need to hide the password in another type of event that also has it), we will have to extract this into a function, but also call it repeatedly from multiple classes, meaning that we are not reusing that much code after all.
- **Boilerplate**: The ``serialize()`` method will have to be present in all event classes, calling the same code. Although we can extract this into another class (creating a mixin), it does not seem like a good use of inheritance.

An alternative solution is to be able to dynamically construct an object that, given a set of
filters (transformation functions) and an event instance, is able to serialize it by applying
the filters to its fields. We then only need to define the functions to transform each type of
field, and the serializer is created by composing many of these functions.

Once we have this object, we can decorate the class in order to add the ``serialize()``
method, which will just call these Serialization objects with itself:

.. code-block:: python

    def hide_field(field) -> str:
        return "**redacted**"

    def format_time(field_timestamp: datetime) -> str:
        return field_timestamp.strftime("%Y-%m-%d %H:%M")

    def show_original(event_field):
        return event_field

    class EventSerializer:
        def __init__(self, serialization_fields: dict) -> None:
            self.serialization_fields = serialization_fields

        def serialize(self, event) -> dict:
            return {
                field: transformation(getattr(event, field))
                for field, transformation in
                self.serialization_fields.items()
            }
    class Serialization:
        def __init__(self, **transformations):
            self.serializer = EventSerializer(transformations)

        def __call__(self, event_class):
            def serialize_method(event_instance):
                return self.serializer.serialize(event_instance)

            event_class.serialize = serialize_method
        return event_class

    @Serialization(
        username=show_original,
        password=hide_field,
        ip=show_original,
        timestamp=format_time
    )
    class LoginEvent:
        def __init__(self, username, password, ip, timestamp):
            self.username = username
            self.password = password
            self.ip = ip
            self.timestamp = timestamp

Notice how the decorator makes it easier for the user to know how each field is going to be
treated without having to look into the code of another class. Just by reading the arguments
passed to the class decorator, we know that the username and IP address will be left
unmodified, the password will be hidden, and the timestamp will be formatted.

Now, the code of the class does not need the ``serialize()`` method defined, nor does it
need to extend from a mixin that implements it, since the decorator will add it. In fact, this
is probably the only part that justifies the creation of the class decorator, because otherwise,
the ``Serialization`` object could have been a class attribute of ``LoginEvent``, but the fact
that it is altering the class by adding a new method to it makes it impossible.

Moreover, we could have another class decorator that, just by defining the attributes of the
class, implements the logic of the init method, but this is beyond the scope of this
example. This is what libraries such as ``attrs`` do, and a similar functionality is
proposed in for the Standard library.

By using this class decorator, the previous example could be
rewritten in a more compact way, without the boilerplate code of the ``init``, as shown here:

.. code-block:: python

    from dataclasses import dataclass
    from datetime import datetime

    @Serialization(
        username=show_original,
        password=hide_field,
        ip=show_original,
        timestamp=format_time
    )
    @dataclass
    class LoginEvent:
        username: str
        password: str
        ip: str
        timestamp: datetime

Note that ``@dataclass`` is a decorator that is used to add generated special methods to classes.
It examines the class to find fields. A field is defined as class variable that has a type annotation.
Nothing in ``dataclass()`` examines the type specified in the variable annotation.

1.3. Other types of decorator
+++++++++++++++++++++++++++++

Now that we know what the ``@`` syntax for decorators actually means, we can conclude that
it isn't just functions, methods, or classes that can be decorated; actually, anything that can
be defined, such as generators, coroutines, and even objects that have already been
decorated, can be decorated, meaning that decorators can be stacked.

The previous example showed how decorators can be chained. We first defined the class,
and then applied ``@dataclass`` to it, which converted it into a data class, acting as a
container for those attributes. After that, the ``@Serialization`` will apply the logic to that
class, resulting in a new class with the new ``serialize()`` method added to it.
Another good use of decorators is for generators that are supposed to be used as
coroutines. The main idea is that, before sending any data to a newly created generator,
the latter has to be advanced up to their next ``yield`` statement by calling ``next()`` on it. This
is a manual process that every user will have to remember and hence is error-prone. We
could easily create a decorator that takes a generator as a parameter, calls ``next()`` to it, and
then returns the generator.

1.4. Passing arguments to decorators
++++++++++++++++++++++++++++++++++++

At this point, we already regard decorators as a powerful tool in Python. However, they
could be even more powerful if we could just pass parameters to them so that their logic is
abstracted even more.

There are several ways of implementing decorators that can take arguments, but we will go
over the most common ones. The first one is to create decorators as nested functions with a
new level of indirection, making everything in the decorator fall one level deeper. The
second approach is to use a class for the decorator.

In general, the second approach favors readability more, because it is easier to think in
terms of an object than three or more nested functions working with closures. However, for
completeness, we will explore both, and the reader can decide what is best for the problem
at hand.

1.4.1. Decorators with nested functions
---------------------------------------

Roughly speaking, the general idea of a decorator is to create a function that returns a
function (often called a higher-order function). The internal function defined in the body of
the decorator is going to be the one actually being called.

Now, if we wish to pass parameters to it, we then need another level of indirection. The
first one will take the parameters, and inside that function, we will define a new function,
which will be the decorator, which in turn will define yet another new function, namely the
one to be returned as a result of the decoration process. This means that we will have at
least three levels of nested functions.

Don't worry if this didn't seem clear so far. After reviewing the examples that are about to
come, everything will become clear.

One of the first examples we saw of decorators implemented the retry functionality over
some functions. This is a good idea, except it has a problem; our implementation did not
allow us to specify the numbers of retries, and instead, this was a fixed number inside the
decorator.

Now, we want to be able to indicate how many retries each instance is going to have, and
perhaps we could even add a default value to this parameter. In order to do this, we need
another level of nested functions—first for the parameters, and then for the decorator itself.
This is because we are now going to have something in the form of the following:
``@retry(arg1, arg2,... )``. And that has to return a decorator because the ``@`` syntax will apply the result
of that computation to the object to be decorated. Semantically, it would translate to something
like the following: ``<original_function> = retry(arg1, arg2, ....)(<original_function>)``

Besides the number of desired retries, we can also indicate the types of exception we wish
to control. The new version of the code supporting the new requirements might look like
this:

.. code-block:: python

    RETRIES_LIMIT = 3

    def with_retry(retries_limit=RETRIES_LIMIT, allowed_exceptions=None):
        allowed_exceptions = allowed_exceptions or (ControlledException,)

        def retry(operation):
            @wraps(operation)
            def wrapped(*args, **kwargs):
                last_raised = None
                for _ in range(retries_limit):
                    try:
                        return operation(*args, **kwargs)
                    except allowed_exceptions as e:
                        logger.info("retrying %s due to %s", operation, e)
                        last_raised = e
                raise last_raised
            return wrapped
        return retry

Here are some examples of how this decorator can be applied to functions, showing the
different options it accepts:

.. code-block:: python

    @with_retry()
    def run_operation(task):
        return task.run()

    @with_retry(retries_limit=5)
    def run_with_custom_retries_limit(task):
        return task.run()

    @with_retry(allowed_exceptions=(AttributeError,))
    def run_with_custom_exceptions(task):
        return task.run()

    @with_retry(
        retries_limit=4, allowed_exceptions=(ZeroDivisionError, AttributeError)
    )
    def run_with_custom_parameters(task):
        return task.run()

1.4.2. Decorator objects
------------------------

The previous example requires three levels of nested functions. The first it is going to be a
function that receives the parameters of the decorator we want to use. Inside this function,
the rest of the functions are closures that use these parameters along with the logic of the
decorator.

A cleaner implementation of this would be to use a class to define the decorator. In this
case, we can pass the parameters in the ``__init__`` method, and then implement the logic of
the decorator on the magic method named ``__call__``.

The code for the decorator will look like it does in the following example:

.. code-block:: python

    class WithRetry:
        def __init__(self, retries_limit=RETRIES_LIMIT,
            allowed_exceptions=None):
            self.retries_limit = retries_limit
            self.allowed_exceptions = allowed_exceptions or (ControlledException,)

        def __call__(self, operation):
            @wraps(operation)
            def wrapped(*args, **kwargs):
                last_raised = None
                for _ in range(self.retries_limit):
                    try:
                        return operation(*args, **kwargs)
                    except self.allowed_exceptions as e:
                        logger.info("retrying %s due to %s", operation, e)
                        last_raised = e

                raise last_raised

            return wrapped

And this decorator can be applied pretty much like the previous one, like so:

.. code-block:: python

    @WithRetry(retries_limit=5)
    def run_with_custom_retries_limit(task):
        return task.run()

It is important to note how the Python syntax takes effect here. First, we create the object, so
before the ``@`` operation is applied, the object is created with its parameters passed to it. This
will create a new object and initialize it with these parameters, as defined in the ``init``
method. After this, the ``@`` operation is invoked, so this object will wrap the function named
``run_with_custom_retries_limit``, meaning that it will be passed to the call magic
method.

Inside this ``call`` magic method, we defined the logic of the decorator as we normally
do: we wrap the original function, returning a new one with the logic we want instead.

1.5. Good uses for decorators
+++++++++++++++++++++++++++++

In this section, we will take a look at some common patterns that make good use of
decorators. These are common situations for when decorators are a good choice.

From all the countless applications decorators can be used for, we will enumerate a few, the
most common or relevant:

- **Transforming parameters**: Changing the signature of a function to expose a nicer API, while encapsulating details on how the parameters are treated and transformed underneath.
- **Tracing code**: Logging the execution of a function with its parameters.
- **Validate parameters**.
- **Implement retry operations**.
- **Simplify classes by moving some (repetitive) logic into decorators**.

1.5.1. Transforming parameters
------------------------------

We have mentioned before that decorators can be used to validate parameters (and even
enforce some preconditions or postconditions under the idea of DbC), so from this you
probably have got the idea that it is somehow common to use decorators when dealing
with or manipulating parameters.

In particular, there are some cases on which we find ourselves repeatedly creating similar
objects, or applying similar transformations that we would wish to abstract away. Most of
the time, we can achieve this by simply using a decorator.

1.5.2. Tracing code
-------------------

When talking about **tracing** in this section, we will refer to something more general that has
to do with dealing with the execution of a function that we wish to monitor. This could
refer to scenarios in which we want to:

- Actually trace the execution of a function (for example, by logging the lines it executes)
- Monitor some metrics over a function (such as CPU usage or memory footprint)
- Measure the running time of a function
- Log when a function was called, and the parameters that were passed to it
