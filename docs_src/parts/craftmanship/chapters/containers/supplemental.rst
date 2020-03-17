4. Supplemental data types and containers
*****************************************

In the previous subsections, we concentrated mostly on those data types that have
dedicated literals in the Python syntax. These were also the types that are implemented at
the interpreter-level. However, Python's standard library offers a great collection of
supplemental data types that can be effectively used in places where the basic built-in types
show their shortcomings, or places where the nature of the data requires specialized
handling (for example, in the presentation of time and dates).

The most common are data containers that are found in the collections, and we have
already briefly mentioned two of them: ``deque`` and ``OrderedDict``. However, the landscape
of data structures available for Python programmers is enormous and almost every module
of the Python standard library defines some specialized types for handling the data of
different problem domains.

4.1. Specialized data containers from the collections module
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Every data structure has its shortcomings. There is no single collection that can suit every
problem, and four basic types of them (tuple, list, set, and dictionary) is still not a wide
range of choices. These are the most basic and important collections that have a dedicated
literal syntax. Fortunately, Python provides far more options in its standard library through
the ``collections`` built-in module. Here are the most important universal data containers
provided by this module:

- ``namedtuple()``: This is a factory function for creating tuple subclasses whose indexes can be accessed as named attributes
- ``deque``: This is a double-ended queue, a list-like generalization of stacks and queues with fast appends and pops on both ends
- ``ChainMap``: This is a dictionary-like class to create a single view of multiple mappings
- ``Counter``: This is a dictionary subclass for counting hashable objects
- ``OrderedDict``: This is a dictionary subclass that preserves the order that the entries were added in
- ``defaultdict``: This is a dictionary subclass that can supply missing values using a user-defined factory function

4.2. Symbolic enumeration with the enum module
++++++++++++++++++++++++++++++++++++++++++++++

One of the special handy types found in the Python standard is the ``Enum`` class from the
``enum`` module. This is a base class that allows you to define symbolic enumerations, similar
in concept to the enumerated types found in many other programming languages (C, C++,
C#, Java, and many more) that are often denoted with the ``enum`` keyword.

In order to define your own enumeration in Python, you will need to subclass the ``Enum``
class and define all enumeration members as class attributes. The following is an example
of a simple Python ``enum``:

.. code-block:: python

    from enum import Enum

    class Weekday(Enum):
        MONDAY = 0
        TUESDAY = 1
        WEDNESDAY = 2
        THURSDAY = 3
        FRIDAY = 4
        SATURDAY = 5
        SUNDAY = 6

The Python documentation defines the following nomenclature for ``enum``:

- ``enumeration`` or ``enum``: This is the subclass of ``Enum`` base class. Here, it would be ``Weekday``.
- ``member``: This is the attribute you define in the Enum subclass. Here, it would be ``Weekday.MONDAY``, ``Weekday.TUESDAY``, and so on.
- ``name``: This is the name of the ``Enum`` subclass attribute that defines the member. Here, it would be ``MONDAY`` for ``Weekday.MONDAY``, ``TUESDAY`` for ``Weekday.TUESDAY``, and so on.
- ``value``: This is the value assigned to the Enum subclass attribute that defines the ``member``. Here, for ``Weekday.MONDAY`` it would be one, for ``Weekday.TUESDAY`` it would be two, and so on.

You can use any type as the ``enum`` member value. If the member value is not important in
your code, you can even use the ``auto()`` type, which will be replaced with automatically
generated values. Here is the previous example rewritten with the use of ``auto`` in it:

.. code-block:: python

    from enum import Enum, auto


    class Weekday(Enum):
         MONDAY = auto()
         TUESDAY = auto()
         WEDNESDAY = auto()
         THURSDAY = auto()
         FRIDAY = auto()
         SATURDAY = auto()
         SUNDAY = auto()

Enumerations in Python are really useful in every place where some variable can take a
finite number of values/choices. For instance, they can be used to define statues of objects,
as shown in the following example:

.. code-block:: python

    from enum import Enum, auto


    class OrderStatus(Enum):
         PENDING = auto()
         PROCESSING = auto()
         PROCESSED = auto()

    class Order:
         def __init__(self):
             self.status = OrderStatus.PENDING

         def process(self):
             if self.status == OrderStatus.PROCESSED:
                raise RuntimeError("Can't process order that has been already processed")

             self.status = OrderStatus.PROCESSING
             ...
             self.status = OrderStatus.PROCESSED

Another use case for enumerations is storing selections of non-exclusive choices. This is
something that is often implemented using bit flags and bit masks in languages where bit
manipulation of numbers is very common, like C. In Python, this can be done in a more
expressive and convenient way using ``FlagEnum``:

.. code-block:: python

    from enum import Flag, auto


    class Side(Flag):
         GUACAMOLE = auto()
         TORTILLA = auto()
         FRIES = auto()
         BEER = auto()
         POTATO_SALAD = auto()

You can combine such flags using bitwise operations (the ``|`` and ``&`` operators) and test for
flag membership with the ``in`` keyword. Here are some examples for a ``Side`` enumeration:

.. code-block:: python

    >>> mexican_sides = Side.GUACAMOLE | Side.BEER | Side.TORTILLA
    >>> bavarian_sides = Side.BEER | Side.POTATO_SALAD
    >>> common_sides = mexican_sides & bavarian_sides
    >>> Side.GUACAMOLE in mexican_sides
    True
    >>> Side.TORTILLA in bavarian_sides
    False
    >>> common_sides
    <Side.BEER: 8>

Symbolic enumerations share some similarity with dictionaries and named tuples because
they all map names/keys to values. The main difference is that the ``Enum`` definition is
immutable and global. It should be used whenever there is a closed set of possible values
that can't change dynamically during program runtime, and especially if that set should be
defined only once and globally. Dictionaries and named tuples are data containers. You can
create as many instances of them as you like.
