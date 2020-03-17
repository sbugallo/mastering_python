1. Underscores in Python
************************

Consider the following example to illustrate this:

.. code-block:: python

    class Connector:

        def __init__(self, source, user, password, timeout):
            self.source = source
            self.user = user
            self.__password = password
            self._timeout = timeout

.. code-block:: python

    >>> Connector(...).source
    'postgresql://localhost'

    >>> Connector(...)._timeout
    60

    >>> Connector(...).__password
    Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
    AttributeError: 'Connector' object has no attribute '__password'

    >>> vars(Connector(...))
    {'source': 'postgresql://localhost', '_timeout': 60, 'user': 'root', '_Connector__password': '1234'}

Here, a Connector object is created with source, and it starts with 4 attributes—the
aforementioned ``source``, ``timeout``, ``user`` and ``password``. ``source`` and ``user`` are public, ``timeout`` is
private and ``password`` is.

However, as we can see from the following lines when we create an object like this, we can actually access ``timeout``.
The interpretation of this code is that ``_timeout`` should be accessed only within connector itself and never from a
caller. This means that you should organize the code in a way so that you can safely refactor the timeout at all of the
times it's needed, relying on the fact that it's not being called from outside the object (only internally), hence
preserving the same interface as before. Complying with these rules makes the code easier to maintain and more
robust because we don't have to worry about ripple effects when refactoring. The same principle applies to methods as
well.

.. note::

    Objects should only expose those attributes and methods that are relevant to an external caller object,
    namely, entailing its interface. Everything that is not strictly part of an object's interface should be kept prefixed
    with a single underscore.

This is the Pythonic way of clearly delimiting the interface of an object. There is, however, a common misconception
that some attributes and methods can be actually made private. This is, again, a misconception.

``password`` is defined with a double underscore instead. Some developers use this method to hide some attributes,
thinking, like in this example, that ``password`` is now private and that no other object can modify it. Now, take a
look at the exception that is raised when trying to access it. It's ``AttributeError``, saying that it doesn't exist.
It doesn't say something like "this is private" or "this can't be accessed" and so on. It says it does not exist. This
should give us a clue that, in fact, something different is happening and that this behavior is instead just a side
effect, but not the real effect we want.

What's actually happening is that with the double underscores, Python creates a different name for the attribute (this
is called **name mangling**). What it does is create the attribute with the following name instead:
"_<class-name>__<attribute-name>". In this case, an attribute named '_Connector__password' will be created.

Notice the side effect that we mentioned earlier—the attribute only exists with a different name, and for that reason
the AttributeError was raised on our first attempt to access it.

The idea of the double underscore in Python is completely different. It was created as a means to override different
methods of a class that is going to be extended several times, without the risk of having collisions with the method
names. Even that is a too far-fetched use case as to justify the use of this mechanism.

Double underscores are a non-Pythonic approach. If you need to define attributes as private, use a single underscore,
and respect the Pythonic convention that it is a private attribute.

.. note:: Do not use double underscores.
