3. Properties, attributes and different types of methods for objects
********************************************************************

All of the properties and functions of an object are public in Python, which is different from other languages where
properties can be public, private, or protected. That is, there is no point in preventing caller objects from invoking
any attributes an object has. This is another difference with respect to other programming languages in which you can
mark some attributes as private or protected.

There is no strict enforcement, but there are some conventions. An attribute that starts with an underscore is meant to
be private to that object, and we expect that no external agent calls it (but again, there is nothing preventing this).

3.1. Underscores in Python
++++++++++++++++++++++++++

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

3.2 Properties
++++++++++++++

When the object needs to just hold values, we can use regular attributes. Sometimes, we might want to do some
computations based on the state of the object and the values of other attributes. Most of the time, properties are a
good choice for this.

Properties are to be used when we need to define access control to some attributes in an object, which is another point
where Python has its own way of doing things. In other programming languages (like Java), you would create access
methods (getters and setters), but idiomatic Python would use properties instead.

Imagine that we have an application where users can register and we want to protect certain information about the user
from being incorrect, such as their email, as shown in the following code:

.. code-block:: python

    import re

    EMAIL_FORMAT = re.compile(r"[^@]+@[^@]+\.[^@]+")

    def is_valid_email(potentially_valid_email: str):
        return re.match(EMAIL_FORMAT, potentially_valid_email) is not None

    class User:
         def __init__(self, username):
             self.username = username
             self._email = None

         @property
         def email(self):
            return self._email

         @email.setter
         def email(self, new_email):
             if not is_valid_email(new_email):
                raise ValueError(f"Can't set {new_email} as it's not a valid email")
             self._email = new_email


By putting ``email`` under a property, we obtain some advantages for free. In this example, the first ``@property``
method will return the value held by the private attribute ``email``. As mentioned earlier, the leading underscore
determines that this attribute is intended to be used as private, and therefore should not be accessed from outside this
class.

Then, the second method uses ``@email.setter``, with the already defined property of the previous method. This is the
one that is going to be called when ``<user>.email = <new_email>`` runs from the caller code, and ``<new_email>`` will
become the parameter of this method. Here, we explicitly defined a validation that will fail if the value that is trying
to be set is not an actual email address. If it is, it will then update the attribute with the new value as follows:

.. code-block:: python

    >>> u1 = User("jsmith")
    >>> u1.email = "jsmith@"
    Traceback (most recent call last):
    ...
    ValueError: Can't set jsmith@ as it's not a valid email
    >>> u1.email = "jsmith@g.co"
    >>> u1.email
    'jsmith@g.co'

This approach is much more compact than having custom methods prefixed with ``get_`` or ``set_``. It's clear what is
expected because it's just email.

.. note::

    Don't write custom ``get_*`` and ``set_*`` methods for all attributes on your objects. Most of the time, leaving
    them as regular attributes is just enough. If you need to modify the logic for when an attribute is retrieved or
    modified, then use properties.

You might find that properties are a good way to achieve command and query separation (CC08). Command and query
separation state that a method of an object should either answer to something or do something, but not both. If a method
of an object is doing something and at the same time it returns a status answering a question of how that operation
went, then it's doing more than one thing, clearly violating the principle that functions should do one thing, and one
thing only.

Depending on the name of the method, this can create even more confusion, making it harder for readers to understand
what the actual intention of the code is. For example, if a method is called set_email, and we use it as
if self.set_email("a@j.com"): ..., what is that code doing? Is it setting the email to a@j.com? Is it checking if the
email is already set to that value? Both (setting and then checking if the status is correct)?

With properties, we can avoid this kind of confusion. The ``@property`` decorator is the query that will answer to
something, and the ``@<property_name>.setter`` is the command that will do something.

Another piece of good advice derived from this example is as follows: don't do more than one thing on a method. If you
want to assign something and then check the value, break that down into two or more sentences.

.. note::

    Methods should do one thing only. If you have to run an action and then check for the status, so that in separate
    methods that are called by different statements.

