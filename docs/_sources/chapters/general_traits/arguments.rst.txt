6. Arguments in functions and methods
*************************************

In Python, functions can be defined to receive arguments in several different ways, and these arguments can
also be provided by callers in multiple ways.

There is also an industry-wide set of practices for defining interfaces in software engineering that closely
relates to the definition of arguments in functions.

6.1. How function arguments work in Python
++++++++++++++++++++++++++++++++++++++++++

First, we will explore the particularities of how arguments are passed to functions in Python, and then we
will review the general theory of good software engineering practices that relate to these concepts.

By first understanding the possibilities that Python offers for handling parameters, we will be able to
assimilate general rules more easily, and the idea is that after having done so, we can easily draw
conclusions on what good patterns or idioms are when handling arguments. Then, we can identify in which
scenarios the Pythonic approach is the correct one, and in which cases we might be abusing the features of the
language.

6.1.1. How arguments are copied to functions
--------------------------------------------

The first rule in Python is that all arguments are passed by a value. Always. This means that when passing
values to functions, they are assigned to the variables on the signature definition of the function to be
later used on it. You will notice that a function changing arguments might depend on the type arguments: if we
are passing mutable objects, and the body of the function modifies this, then, of course, we have side-effect
that they will have been changed by the time the function returns.

In the following we can see the difference:

.. code-block:: python

    >>> def function(argument):
    ...     argument += " in function"
    ...     print(argument)
    ...
    >>> immutable = "hello"
    >>> function(immutable)
    hello in function
    >>> mutable = list("hello")
    >>> immutable
    'hello'
    >>> function(mutable)
    ['h', 'e', 'l', 'l', 'o', ' ', 'i', 'n', ' ', 'f', 'u', 'n', 'c', 't', 'i', 'o', 'n']
    >>> mutable
    ['h', 'e', 'l', 'l', 'o', ' ', 'i', 'n', ' ', 'f', 'u', 'n', 'c', 't', 'i', 'o', 'n']

This might look like an inconsistency, but it's not. When we pass the first argument, a string, this is
assigned to the argument on the function. Since string objects are immutable, a statement like
``argument += <expression>`` will in fact create the new object, ``argument + <expression>``, and assign
that back to the argument. At that point, an argument is just a local variable inside the scope of the
function and has nothing to do with the original one in the caller.

On the other hand, when we pass list, which is a mutable object, then that statement has a different meaning
(it's actually equivalent to calling ``.extend()`` on that list). This operator acts by modifying the list
in-place over a variable that holds a reference to the original list object, hence modifying it.

We have to be careful when dealing with these types of parameter because it can lead to unexpected
side-effects. Unless you are absolutely sure that it is correct to manipulate mutable arguments in this way,
we would recommend avoiding it and going for alternatives without these problems.

.. note:: Don't mutate function arguments. In general, try to avoid side-effects in functions as much as possible.

Arguments in Python can be passed by position, as in many other programming languages, but also by keyword.
This means that we can explicitly tell the function which values we want for which of its parameters. The only
caveat is that after a parameter is passed by keyword, the rest that follow must also be passed this way,
otherwise, SyntaxError will be raised.

6.1.2. Variable number of arguments
-----------------------------------

Python, as well as other languages, has built-in functions and constructions that can take a variable number
of arguments. Consider for example string interpolation functions (whether it be by using the % operator or
the format method for strings), which follow a similar structure to the printf function in C, a first
positional parameter with the string format, followed by any number of arguments that will be placed on the
markers of that formatting string.

Besides taking advantage of these functions that are available in Python, we can also create our own, which
will work in a similar fashion. In this section, we will cover the basic principles of functions with a
variable number of arguments, along with some recommendations, so that in the next section, we can explore how
to use these features to our advantage when dealing with common problems, issues, and constraints that
functions might have if they have too many arguments.

For a variable number of positional arguments, the star symbol (``*``) is used, preceding the name of the
variable that is packing those arguments. This works through the packing mechanism of Python.

Let's say there is a function that takes three positional arguments. In one part of the code, we conveniently
happen to have the arguments we want to pass to the function inside a list, in the same order as they are
expected by the function. Instead of passing them one by one by the position (that is, ``list[0]`` to the
first element, ``list[1]`` to the second, and so on), which would be really un-Pythonic, we can use the
packing mechanism and pass them all together in a single instruction:

.. code-block:: python

    >>> def f(first, second, third):
    ...     print(first)
    ...     print(second)
    ...     print(third)
    ...
    >>> l = [1, 2, 3]
    >>> f(*l)
    1
    2
    3

The nice thing about the packing mechanism is that it also works the other way around. If we want to extract
the values of a list to variables, by their respective position, we can assign them like this:

.. code-block:: python

    >>> a, b, c = [1, 2, 3]
    >>> a
    1
    >>> b
    2
    >>> c
    3

Partial unpacking is also possible. Let's say we are just interested in the first values of a sequence (this
can be a list, tuple, or something else), and after some point we just want the rest to be kept together. We
can assign the variables we need and leave the rest under a packaged list. The order in which we unpack is not
limited. If there is nothing to place in one of the unpacked subsections, the result will be an empty list:

.. code-block:: python

    >>> def show(e, rest):
    ...     print("Element: {0} - Rest: {1}".format(e, rest))
    ...
    >>> first, *rest = [1, 2, 3, 4, 5]
    >>> show(first, rest)
    Element: 1 - Rest: [2, 3, 4, 5]
    >>> *rest, last = range(6)
    >>> show(last, rest)
    Element: 5 - Rest: [0, 1, 2, 3, 4]
    >>> first, *middle, last = range(6)
    >>> first
    0
    >>> middle
    [1, 2, 3, 4]
    >>> last
    5
    >>> first, last, *empty = (1, 2)
    >>> first
    1
    >>> last
    2
    >>> empty
    []

One of the best uses for unpacking variables can be found in iteration. When we have to iterate over a
sequence of elements, and each element is, in turn, a sequence, it is a good idea to unpack at the same time
each element is being iterated over. To see an example of this in action, we are going to pretend that we have
a function that receives a list of database rows, and that it is in charge of creating users out of that data.
The first implementation takes the values to construct the user with from the position of each column in the
row, which is not idiomatic at all. The second implementation uses unpacking while iterating:

.. code-block:: python

    USERS = [(i, f"first_name_{i}", "last_name_{i}") for i in range(1_000)]

    class User:
        def __init__(self, user_id, first_name, last_name):
            self.user_id = user_id
            self.first_name = first_name
            self.last_name = last_name

        def bad_users_from_rows(dbrows) -> list:
            """A bad case (non-pythonic) of creating ``User``s from DB rows."""
            return [User(row[0], row[1], row[2]) for row in dbrows]

        def users_from_rows(dbrows) -> list:
        """Create ``User``s from DB rows."""
        return [User(user_id, first_name, last_name) for (user_id, first_name, last_name) in dbrows]

Notice that the second version is much easier to read. In the first version of the function
(``bad_users_from_rows``), we have data expressed in the form ``row[0]``, ``row[1]``, and ``row[2]``, which
doesn't tell us anything about what they are. On the other hand, variables such as ``user_id``, ``first_name``,
and ``last_name`` speak for themselves.

We can leverage this kind of functionality to our advantage when designing our own functions.

An example of this that we can find in the standard library lies in the max function, which is defined as
follows:

.. code-block:: python

    max(...)
    max(iterable, *[, default=obj, key=func]) -> value
    max(arg1, arg2, *args, *[, key=func]) -> value

With a single iterable argument, return its biggest item. The default keyword-only argument specifies an
object to return if the provided iterable is empty.

With two or more arguments, return the largest argument.

There is a similar notation, with two stars ( ** ) for keyword arguments. If we have a dictionary and we pass
it with a double star to a function, what it will do is pick the keys as the name for the parameter, and pass
the value for that key as the value for that parameter in that function.

For instance, check this out:

.. code-block:: python

    function(**{"key": "value"})

It is the same as the following:

.. code-block:: python

    function(key="value")

Conversely, if we define a function with a parameter starting with two-star symbols, the opposite will happen:
keyword-provided parameters will be packed into a dictionary:

.. code-block:: python

    >>> def function(**kwargs):
    ...     print(kwargs)
    ...
    >>> function(key="value")
    {'key': 'value'}

6.2. The number of arguments in functions
+++++++++++++++++++++++++++++++++++++++++

Having functions or methods that take too many arguments is a sign of bad design (a code smell). Then, we
propose ways of dealing with this issue.

The first alternative is a more general principle of software design: **reification** (creating a new object
for all of those arguments that we are passing, which is probably the abstraction we are missing). Compacting
multiple arguments into a new object is not a solution specific to Python, but rather something that we can
apply in any programming language.

Another option would be to use the Python-specific features we saw in the previous section, making use of
variable positional and keyword arguments to create functions that have a dynamic signature. While this might
be a Pythonic way of proceeding, we have to be careful not to abuse the feature, because we might be creating
something that is so dynamic that it is hard to maintain. In this case, we should take a look at the body of
the function. Regardless of the signature, and whether the parameters seem to be correct, if the function is
doing too many different things responding to the values of the parameters, then it is a sign that it has to
be broken down into multiple smaller functions (remember, functions should do one thing, and one thing only!).

6.2.1. Function arguments and coupling
--------------------------------------

The more arguments a function signature has, the more likely this one is going to be tightly coupled with the
caller function.

Let's say we have two functions, ``f1``, and ``f2``, and the latter takes five parameters. The more parameters
``f2`` takes, the more difficult it would be for anyone trying to call that function to gather all that
information and pass it along so that it can work properly.

Now, ``f1`` seems to have all of this information because it can call it correctly. From this, we can derive
two conclusions: first, ``f2`` is probably a leaky abstraction, which means that since ``f1`` knows everything
that ``f2`` requires, it can pretty much figure out what it is doing internally and will be able to do it by
itself. So, all in all, ``f2`` is not abstracting that much. Second, it looks like ``f2`` is only useful to
``f1``, and it is hard to imagine using this function in a different context, making it harder to reuse.

When functions have a more general interface and are able to work with higher-level abstractions, they become
more reusable.

This applies to all sort of functions and object methods, including the ``__init__`` method for classes. The
presence of a method like this could generally (but not always) mean that a new higher-level abstraction
should be passed instead, or that there is a missing object.

.. note:: If a function needs too many parameters to work properly, consider it a code smell.

In fact, this is such a design problem that static analysis tools will, by default, raise a warning about
when they encounter such a case. When this happens, don't suppress the warning, refactor it instead.

6.2.2. Compact function signatures that take too many arguments
---------------------------------------------------------------

Suppose we find a function that requires too many parameters. We know that we cannot leave the code base like
that, and a refactor is imperative. But, what are the options? Depending on the case, some of the following
rules might apply. This is by no means extensive, but it does provide an idea of how to solve some scenarios
that occur quite often.

Sometimes, there is an easy way to change parameters if we can see that most of them belong to a common
object. For example, consider a function call like this one:

.. code-block:: python

    track_request(request.headers, request.ip_addr, request.request_id)

Now, the function might or might not take additional arguments, but something is really obvious here: all of
the parameters depend upon ``request``, so why not pass the request object instead? This is a simple change,
but it significantly improves the code. The correct function call should be ``track_request(request)``: not to
mention that, semantically, it also makes much more sense.

While passing around parameters like this is encouraged, in all cases where we pass mutable objects to
functions, we must be really careful about side-effects. The function we are calling should not make any
modifications to the object we are passing because that will mutate the object, creating an undesired
side-effect. Unless this is actually the desired effect (in which case, it must be made explicit), this kind
of behavior is discouraged. Even when we actually want to change something on the object we are dealing with,
a better alternative would be to copy it and return a (new) modified version of it.

.. note:: Work with immutable objects, and avoid side-effects as much as possible.

This brings us to a similar topic: grouping parameters. In the previous example, the parameters were already
grouped, but the group (in this case, the request object) was not being used. But other cases are not as
obvious as that one, and we might want to group all the data in the parameters in a single object that acts
as a container. Needless to say, this grouping has to make sense. The idea here is to reify: create the
abstraction that was missing from our design.

If the previous strategies don't work, as a last resort we can change the signature of the function to accept
a variable number of arguments. If the number of arguments is too big, using ``*args`` or ``**kwargs`` will
make things harder to follow, so we have to make sure that the interface is properly documented and correctly
used, but in some cases this is worth doing.

It's true that a function defined with ``*args`` and ``**kwargs`` is really flexible and adaptable, but the
disadvantage is that it loses its signature, and with that, part of its meaning, and almost all of its
legibility. We have seen examples of how names for variables (including function arguments) make the code much
easier to read. If a function will take any number of arguments (positional or keyword), we might find out
that when we want to take a look at that function in the future, we probably won't know exactly what it was
supposed to do with its parameters, unless it has a very good docstring.
