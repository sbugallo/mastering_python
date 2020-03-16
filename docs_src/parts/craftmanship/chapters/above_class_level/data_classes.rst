2. Data classes
***************

Before we dive deeper into details of Python classes, we will take a small detour. We will
discuss a relatively new addition to the Python language, which are data classes. The
``dataclasses`` module, introduced in Python 3.7, provides a decorator and function that
allows you to easily add generated special methods to your own classes.

Consider the following example. We are building a program that does some geometric
computation and want to have a class that allows us to hold information about two-
dimensional vectors. We will display the data of the vectors on the screen and perform
common mathematical operations, such as addition, subtraction, and equality comparison.
We already know that we can use special methods to achieve that goal. We can
implement our ``Vector`` class as follows:

.. code-block:: python

    class Vector:
        def __init__(self, x, y):
            self.x = x
            self.y = y

        def __add__(self, other):
            """Add two vectors using + operator"""
            return Vector(
                self.x + other.x,
                self.y + other.y
            )

        def __sub__(self, other):
            """Subtract two vectors using - operator"""
            return Vector(
                self.x - other.x,
                self.y - other.y
            )

        def __repr__(self):
            """Return textual representation of vector"""
            return f"<Vector: x={self.x}, y={self.y}>"

        def __eq__(self, other):
            """Compare two vectors for equality"""
            return self.x == other.x and self.y == other.y

The following is the interactive session example that shows how it behaves when used with
common operators:

.. code-block:: python

    >>> Vector(2, 3)
    <Vector: x=2, y=3)
    >>> Vector(5, 3) + Vector(1, 2)
    <Vector: x=6, y=5>
    >>> Vector(5, 3) - Vector(1, 2)
    <Vector: x=4, y=1>
    >>> Vector(1, 1) == Vector(2, 2)
    False
    >>> Vector(2, 2) == Vector(2, 2)
    True

The preceding vector implementation is quite simple, but involves a lot of repetitive code
that could be avoided. If your program uses many similar simple classes that do not require
complex initialization, you'll end up writing a lot of boilerplate code just for
the ``__init__()``, ``__repr__()``, and ``__eq__()`` methods.

With the ``dataclasses`` module, we can make our ``Vector`` class code a lot shorter:

.. code-block:: python

    from dataclasses import dataclass


    @dataclass
    class Vector:
        x: int
        y: int

        def __add__(self, other):
            """Add two vectors using + operator"""
            return Vector(
                self.x + other.x,
                self.y + other.y,
            )

        def __sub__(self, other):
            """Subtract two vectors using - operator"""
            return Vector(
                self.x - other.x,
                self.y - other.y,
            )

The dataclass class decorator reads annotations of the Vector class attribute and
automatically creates the ``__init__()``, ``__repr__()``, and ``__eq__()`` methods. The default
equality comparison assumes that two instances are equal if all their respective attributes
are equal to each other.

But that's not all. Data classes offer many useful features. They can easily be made
compatible with other Python protocols, too. Let's assume we want our ``Vector`` class
instances to be immutable. Thanks to this, they could be used as dictionary keys and as
content sets. You can do this by simply adding a ``frozen=True`` argument to the dataclass
decorator, as in the following example:

.. code-block:: python

    @dataclass(frozen=True)
    class FrozenVector:
        x: int
        y: int

Such a frozen ``Vector`` data class becomes completely immutable, so you won't be able to
modify any of its attributes. You can still add and subtract two Vector instances as in our
example; these operations simply create new ``Vector`` objects.

The final piece of useful information we will cover about data classes in this chapter is that
you can define default values for specific attributes using the ``field()`` constructor. You can
use both static values and constructors of other objects. Consider the following example:

.. code-block:: python

    >>> @dataclass
    ... class DataClassWithDefaults:
    ...     static_default: str = field(default="this is static default value")
    ...     factory_default: list = field(default_factory=list)
    ...
    >>> DataClassWithDefaults()
    DataClassWithDefaults(static_default='this is static default value', factory_default=[])
