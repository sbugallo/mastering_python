6. Container objects
********************

Python provides a good selection of built-in data containers that allow you to efficiently
solve many problems if you choose them wisely. Types that you should already know of
are those that have dedicated literals:

- Lists
- Tuples
- Dictionaries
- Sets

Python is, of course, not limited to these four containers, and it extends the list of possible
choices through its standard library. In many cases, solutions to some problems may be as
simple as making a good choice for the data structure to hold your data.

6.1. Lists and tuples
+++++++++++++++++++++

Two of the most basic collection types in Python are lists and tuples, and they both
represent sequences of objects. The basic difference between them should be obvious for
anyone who has spent more than a few hours with Python; lists are dynamic, so they can
change their size, while tuples are immutable (cannot be modified after they are created).

Lists and tuples in Python have various optimizations that make allocations/deallocations
of small objects fast. They are also the recommended datatypes for structures where the
position of the element is information by itself. For example, a tuple may be a good choice
for storing a pair of (x, y) coordinates. Implementation details regarding tuples are not
interesting. The only important thing about them in the scope of this chapter is
that tuple is immutable and thus hashable. A detailed explanation of this section will be
covered later in the Dictionaries section. More interesting than tuples is its dynamic
counterpart: lists. In the next section, we will discuss how it really works, and how to deal
with it efficiently.

6.1.1. Implementation details
-----------------------------

Many programmers easily confuse Python's ``list`` type with the concept of linked lists
which are found often in standard libraries of other languages, such as C, C++, or Java. In
fact, CPython lists are not lists at all. In CPython, lists are implemented as variable length
arrays. This should be also true for other implementations, such as Jython and IronPython,
although such implementation details are often not documented in these projects. The
reasons for such confusion is clear. This datatype is named ``list`` and also has an interface
that could be expected from any linked list implementation.

Why it is important and what does it mean? Lists are one of the most popular data
structures, and the way in which they are used greatly affects every application's
performance. CPython is the most popular and used implementation, so knowing its
internal implementation details is crucial.

Lists in Python are contiguous arrays of references to other objects. The pointer to this array
and the length is stored in the list's head structure. This means that every time an item is
added or removed, the array of references needs to be resized (reallocated). Fortunately, in
Python, these arrays are created with exponential over allocation, so not every operation
requires an actual resize of the underlying array. This is how the amortized cost of
appending and popping elements can be low in terms of complexity. Unfortunately, some
other operations that are considered cheap in ordinary linked lists have relatively high
computational complexity in Python:

- Inserting an item at an arbitrary place using the ``list.insert`` method has complexity O(n).
- Deleting an item using ``list.delete`` or using the del operator har has complexity O(n)

At least retrieving or setting an element using an index is an operation where cost is
independent of the list's size, and the complexity of these operations is always O(1).

Let's define ``n`` as the length of a list. Here is a full table of average time complexities for
most of the list operations:
====================================  ==========
Operation                             Complexity
====================================  ==========
Copy                                  O(n)
Append                                O(1)
Insert                                O(n)
Get item                              O(1)
Set item                              O(1)
Delete item                           O(n)
Iteration                             O(n)
Get slice of length ``k``             O(k)
Del slice                             O(n)
Set slice of length ``k``             O(k+n)
Extend                                O(k)
Multiply by ``k``                     O(nk)
Test existence (``element in list``)  O(n)
``min()/max()``                       O(n)
Get length                            O(1)
====================================  ==========

For situations where a real linked list or doubly linked list is required, Python provides
a ``deque`` type in the ``collections`` built-in module. This is a data structure that allows us to
append and pop elements at each side with O(1) complexity. This is a generalization of
stacks and queues, and should work fine anywhere where a doubly linked list is required.

6.1.2. List comprehensions
--------------------------

As you probably know, writing a piece of code such as this can be tedious:

.. code-block:: python

    >>> evens = []
    >>> for i in range(10):
    ...     if i % 2 == 0:
    ...         evens.append(i)
    ...
    >>> evens
    [0, 2, 4, 6, 8]

This may work for C, but it actually makes things slower for Python for the following
reasons:

- It makes the interpreter work on each loop to determine what part of the sequence has to be changed
- It makes you keep a counter to track what element has to be processed
- It requires additional function lookups to be performed at every iteration because ``append()`` is a list's method

A list comprehension is a better pattern for these kind of situations. It allows us to define a
list by using a single line of code:

.. code-block:: python

    >>> [i for i in range(10) if i % 2 == 0]
    [0, 2, 4, 6, 8]

This form of writing is much shorter and involves fewer elements. In a bigger program, this
means less bugs and code that is easier to understand. This is the reason why many
experienced Python programmers will consider such forms as being more readable.

.. tip::

    There is a myth among some Python programmers that list
    comprehensions can be a workaround for the fact that the internal array
    representing the list object must be resized with every few additions.
    Some say that the array will be allocated once in just the right size.
    Unfortunately, this isn't true.

    The interpreter, during evaluation of the comprehension, can't know how
    big the resulting container will be, and it can't preallocate the final size of
    the array for it. Due to this, the internal array is reallocated in the same
    pattern as it would be in the for loop. Still, in many cases, list creation
    using comprehensions is both cleaner and faster than using ordinary
    loops.

6.1.3. Other idioms
-------------------

Another typical example of a Python idiom is the use of ``enumerate()``. This built-in
function provides a convenient way to get an index when a sequence is iterated inside of a
loop. Consider the following piece of code as an example of tracking the element index
without the ``enumerate()`` function:

.. code-block:: python

    >>> i = 0
    >>> for element in ['one', 'two', 'three']:
    ...     print(i, element)
    ...     i += 1
    ...
    0 one
    1 two
    2 three

This can be replaced with the following code, which is shorter and definitely cleaner:

.. code-block:: python

    >>> for i, element in enumerate(['one', 'two', 'three']):
    ...     print(i, element)
    ...
    0 one
    1 two
    2 three

If you need to aggregate elements of multiple lists (or any other iterables) in the one-by-one
fashion, you can use the built-in ``zip()``. This is a very common pattern for uniform iteration
over two same-sized iterables:

.. code-block:: python

    >>> for items in zip([1, 2, 3], [4, 5, 6]):
    ...     print(items)
    ...
    (1, 4)
    (2, 5)
    (3, 6)

Note that the results of ``zip()`` can be reversed by another ``zip()`` call:

.. code-block:: python

    >>> for items in zip(*zip([1, 2, 3], [4, 5, 6])):
    ...     print(items)
    ...
    (1, 2, 3)
    (4, 5, 6)

One important thing you need to remember about the ``zip()`` function is that it expects
input iterables to be the same size. If you provide arguments of different lengths, then it
will trim the output to the shortest argument, as shown in the following example:

.. code-block:: python

    >>> for items in zip([1, 2, 3, 4], [1, 2]):
    ...     print(items)
    ...
    (1, 1)
    (2, 2)

Another popular syntax element is sequence unpacking. It is not limited to lists and tuples,
and will work with any sequence type (even strings and byte sequences). It allows us to
unpack a sequence of elements into another set of variables as long as there are as many
variables on the left-hand side of the assignment operator as the number of elements in the
sequence. If you paid attention to the code snippets, then you might have already noticed
this idiom when we were discussing the ``enumerate()`` function.

The following is a dedicated example of that syntax element:

.. code-block:: python

    >>> first, second, third = "foo", "bar", 100
    >>> first
    'foo'
    >>> second
    'bar'
    >>> third
    100

Unpacking also allows us to capture multiple elements in a single variable using starred
expressions as long as it can be interpreted unambiguously. Unpacking can also be
performed on nested sequences. This can come in handy, especially when iterating on some
complex data structures built out of multiple sequences. Here are some examples of more
complex sequence unpacking:

.. code-block:: python

    >>> first, second, *rest = 0, 1, 2, 3
    >>> first
    0
    >>> second
    1
    >>> rest
    [2, 3]

    >>> first, *inner, last = 0, 1, 2, 3
    >>> first
    0
    >>> inner
    [1, 2]
    >>> last
    3

    >>> (a, b), (c, d) = (1, 2), (3, 4)
    >>> a, b, c, d
    (1, 2, 3, 4)

6.2. Dictionaries
+++++++++++++++++

Dictionaries are one of most versatile data structures in Python. The ``dict`` type allows you
to map a set of unique keys to values, as follows:

.. code-block:: python

    {
        1: ' one',
        2: ' two',
        3: ' three'
    }

Dictionary literals are a very basic thing, and you should already know about them. Python
allows programmers to also create a new dictionary using comprehensions, similar to the
list comprehensions mentioned earlier. Here is a very simple example that maps numbers
in a range from 0 to 99 to their squares:

.. code-block:: python

    squares = {number: number**2 for number in range(100)}

What is important is that the same benefits of using list comprehensions apply to dictionary
comprehensions. So, in many cases, they are more efficient, shorter, and cleaner. For more
complex code, when many ``if`` statements or function calls are required to create a
dictionary, the simple ``for`` loop may be a better choice, especially if it improves readability.

For Python programmers new to Python 3, there is one important note about iterating over
dictionary elements. The ``keys()``, ``values()``, and ``items()`` dictionary methods are no
longer return lists. Also, their counterparts, ``iterkeys()``, ``itervalues()``,
and ``iteritems()``, which returned iterators instead, are missing in Python 3. Now,
the ``keys()``, ``values()``, and ``items()`` methods return special view objects:

- ``keys()``: This returns the dict_keys object which provides a view on all keys of the dictionary
- ``values()``: This returns the dict_values object which provides a view on all values of the dictionary
- ``items()``: This returns the dict_items object, providing views on all (key, value) two-tuples of the dictionary

View objects provide a view on the dictionary content in a dynamic way so that every time
the dictionary changes, the views will reflect these changes, as shown in this example:

.. code-block:: python

    >>> person = {'name': 'John', 'last_name': 'Doe'}
    >>> items = person.items()
    >>> person['age'] = 42
    >>> items
    dict_items([('name', 'John'), ('last_name', 'Doe'), ('age', 42)])

View objects join the behavior of lists returned by the implementation of old methods with
iterators that have been returned by their ´´iter´´ counterparts. Views do not need to
redundantly store all values in memory (like lists do), but are still allowed to access their
length (using the ´´len()´´ function) and testing for membership (using the ´´in´´ keyword).
Views are, of course, iterable.

The last important thing about views is that both view objects returned by
the ``keys()`` and ``values()`` methods ensure the same order of keys and values. In Python 2,
you could not modify the dictionary content between these two calls if you wanted to
ensure the same order of retrieved keys and values. ``dict_keys`` and ``dict_values`` are now
dynamic, so even if the content of the dictionary changes between
the ``keys()`` and ``values()`` calls, the order of iteration is consistent between these two
views.

6.2.1. Implementation details
-----------------------------

CPython uses hash tables with pseudo-random probing as an underlying data structure for
dictionaries. It seems like a very deep implementation detail, but it is very unlikely to
change in the near future, so it is also a very interesting fact for the Python programmer.

Due to this implementation detail, only objects that are hashable can be used as a
dictionary key. An object is hashable if it has a hash value that never changes during its
lifetime, and can be compared to different objects. Every Python built-in type that is
immutable is also hashable. Mutable types, such as list, dictionaries, and sets, are not
hashable, and so they cannot be used as dictionary keys. Protocol that defines if a type is
hashable consists of two methods:

- ``__hash__``: This provides the hash value (as an integer) that is needed by the internal ``dict`` implementation. For objects that are instances of user-defined classes, it is derived from their ``id()``.
- ``__eq__``: This compares if two objects have the same value. All objects that are instances of user-defined classes compare as unequal by default, except for themselves.

Two objects that are compared as equal must have the same hash value. The reverse does
not need to be true. This means that collisions of hashes are possible: two objects with the
same hash may not be equal. It is allowed, and every Python implementation must be able
to resolve hash collisions. CPython uses open addressing to resolve them. The
probability of collisions greatly affects dictionary performance, and, if it is high, the
dictionary will not benefit from its internal optimizations.

While three basic operations, adding, getting, and deleting an item, have an average time
complexity equal to O(1), their amortized worst case complexities are a lot higher. It is O(n),
where ``n`` is the current dictionary size. Additionally, if user-defined class objects are used as
dictionary keys and they are hashed improperly (with a high risk of collisions), this will
have a huge negative impact on the dictionary's performance. The full table of CPython's
time complexities for dictionaries is as follows:

=========== ================== ===============================
Operation   Average complexity Amortized worst case complexity
=========== ================== ===============================
Get item    O(1)               O(n)
Set item    O(1)               O(n)
Delete item O(1)               O(n)
Copy        O(n)               O(n)
Iteration   O(n)               O(n)
=========== ================== ===============================

It is also important to know that the ``n`` number in worst case complexities for copying and
iterating the dictionary is the maximum size that the dictionary ever achieved, rather than
the size at the time of operation. In other words, iterating over the dictionary that once was
huge but greatly shrunk in time may take a surprisingly long time. In some cases, it may be
better to create a new dictionary object from a dictionary that needs to be shrunk if it has to
be iterated often instead of just removing elements from it.

6.2.2. Weaknesses and alternatives
----------------------------------

For a very long time, one of the most common pitfalls regarding dictionaries was expecting
that they preserve the order of elements in which new keys were added. The situation has
changed a bit in Python 3.6, and the problem was finally solved in Python 3.7 on the level
of language specification.

But, before we dig deeper into the situation of Python 3.6 and later releases, we need to
make a small detour and examine the problem as if we were still stuck in the past, when the
only Python releases available were older than 3.6. In the past, you could have a situation
where the consecutive dictionary keys also had hashes that were consecutive values too.
And, for a very long time, this was the only situation when you could expect that you
would iterate over dictionary elements in the same order as they were added to the
dictionary. The easiest way to present this is by using integer numbers, as hashes of integer
numbers are the same as their value:

.. code-block:: python

    >>> {number: None for number in range(5)}.keys()
    dict_keys([0, 1, 2, 3, 4])

Using other datatypes that hash differently could show that the order is not preserved.
Here is an example that was executed in CPython 3.5:

.. code-block::

    >>> {str(number): None for number in range(5)}.keys()
    dict_keys(['1', '2', '4', '0', '3'])
    >>> {str(number): None for number in reversed(range(5))}.keys()
    dict_keys(['2', '3', '1', '4', '0'])

As shown in the preceding code, for CPython 3.5 (and also earlier versions), the resulting
order is both dependent on the hashing of the object and also on the order in which the
elements were added. This is definitely not what can be relied on, because it can vary with
different Python implementations.

So, what about Python 3.6 and later releases? Starting from Python 3.6, the CPython
interpreter uses a new compact dictionary representation that has a noticeably smaller
memory footprint and also preserves order as a side effect of that new implementation. In
Python 3.6, the order preserving nature of dictionaries was only an implementation detail,
but in Python 3.7, it has been officially declared in the Python language specification. So,
starting from Python 3.7, you can finally rely on the item insertion order of dictionaries.

In parallel to the CPython implementation of dictionaries, Python 3.6 introduced another
change in the syntax that is related to the order of items in dictionaries. As defined in the
PEP 486 "Preserving the order of ``**kwargs`` in a function" document, the order of keyword
arguments collected using the ``**kwargs`` syntax must be the same as presented in function
call. This behavior can be clearly presented with the following example:

.. code-block:: python

    >>> def fun(**kwargs):
    ...     print(kwargs)
    ...
    >>> fun(a=1, b=2, c=3)
    {'a': 1, 'b': 2, 'c': 3}
    >>> fun(c=1, b=2, a=3)
    {'c': 1, 'b': 2, 'a': 3}

However the preceding changes can be used effectively only in the newest releases of
Python. So, what should you do if you have a library that must work on older versions of
Python too, and some parts of its code requires order-preserving dictionaries? The best
option is to be clear about your expectations regarding dictionary ordering and use a type
that explicitly preserves the order of elements.

Fortunately, the Python standard library provides an ordered dictionary type
called ``OrderedDict`` in the ``collections`` module. The constructor of this type accepts
``iterable`` as the initialization argument. Each element of that argument should be a pair of a
dictionary key and value, as in the following example:

.. code-block:: python

    >>> from collections import OrderedDict
    >>> OrderedDict((str(number), None) for number in range(5)).keys()
    odict_keys(['0', '1', '2', '3', '4'])

It also has some additional features, such as popping items from both ends using
the ``popitem()`` method, or moving the specified element to one of the ends using
the`` move_to_end()`` method. A full reference on that collection is available in the Python
documentation (refer to
`https://docs.python.org/3/library/collections.html <https://docs.python.org/3/library/collections.html>`_). Even if
you target only Python in version 3.7 or newer, which guarantees the preservation of the
item insertion order, the ``OrderedDict type`` is still useful. It allows you to make your
intention clear. If you define your variable with ``OrderedDict`` instead of a plain dict, it
becomes obvious that, in this particular case, the order of inserted items is important.

The last interesting note is that, in very old code bases, you can find ``dict`` as a primitive set
implementation that ensures uniqueness of elements. While this will give proper results,
you should avoid such use of that type unless you target Python versions lower than 2.3.
Using dictionaries in this way is wasteful in terms of resources. Python has a builtin ``set``
type that serves this purpose. In fact, it has very similar internal implementation to
dictionaries in CPython, but offers some additional features, as well as specific set-related
optimizations.

6.3. Sets
+++++++++

Sets are a very robust data structure that are mostly useful in situations where the order of
elements is not as important as their uniqueness. They are also useful if you need to
efficiently check efficiency if the element is contained in a collection. Sets in Python are
generalizations of mathematic sets, and are provided as built-in types in two flavors:

- ``set()``: This is a mutable, non-ordered, finite collection of unique, immutable (hashable) objects
- ``frozenset()``: This is an immutable, hashable, non-ordered collection of unique, immutable (hashable) objects

The immutability of ``frozenset()`` objects makes it possible for them to be included as
dictionary keys and also other ``set()`` and ``frozenset()`` elements. A plain
mutable ``set()`` object cannot be used within another ``set()`` or ``frozenset()``. Attempting
to do so will raise a ``TypeError`` exception, as in the following example:

.. code-block:: python

    >>> set([set([1,2,3]), set([2,3,4])])
    Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
    TypeError: unhashable type: 'set'

On the other hand, the following ``set`` initializations are completely correct, and do not raise
exceptions:

.. code-block:: python

    >>> set([frozenset([1,2,3]), frozenset([2,3,4])])
    {frozenset({1, 2, 3}), frozenset({2, 3, 4})}
    >>> frozenset([frozenset([1,2,3]), frozenset([2,3,4])])
    frozenset({frozenset({1, 2, 3}), frozenset({2, 3, 4})})

Mutable sets can be created in three ways:

- Using a ``set()`` call that accepts optional iterables as the initialization argument, such as ``set([0, 1, 2])``
- Using a set comprehension such as ``{element for element in range(3)}``
- Using set literals such as ``{1, 2, 3}``

Note that using literals and comprehensions for sets requires extra caution, because they
are very similar in form to dictionary literals and comprehensions. Also, there is no literal
for empty set objects: empty curly brackets ``{}`` are reserved for empty dictionary literals.

6.3.1. Implementation details
-----------------------------

Sets in CPython are very similar to dictionaries. As a matter of fact, they are implemented
like dictionaries with dummy values, where only keys are actual collection elements. Sets
also exploit this lack of values in mapping for additional optimizations.

Thanks to this, sets allow very fast additions, deletions, and checks for element existence
with the average time complexity equal to O(1). Still, since the implementation of sets in
CPython relies on a similar hash table structure, the worst case complexity for these
operations is still O(n), where ``n`` is the current size of a set.

Other implementation details also apply. The item to be included in a set must be hashable,
and, if instances of user-defined classes in the set are hashed poorly, this will have a
negative impact on their performance.

Despite their conceptual similarity to dictionaries, sets in Python 3.7 do not preserve the
order of elements in specification, or as a detail of CPython implementation.
Let's take a look at the supplemental data types and containers.

6.4. Supplemental data types and containers
+++++++++++++++++++++++++++++++++++++++++++

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

6.4.1. Specialized data containers from the collections module
--------------------------------------------------------------

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

6.4.2. Symbolic enumeration with the enum module
------------------------------------------------

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

6.5. Custom containers
++++++++++++++++++++++

Containers are objects that implement a ``__contains__`` method (that usually returns a Boolean value). This method is
called in the presence of the ``in`` keyword of Python. Something like ``element in container`` becomes
``container.__contains__(element)``.

You can imagine how much more readable and Pythonic the code can be when this method is properly implemented.

Let's say we have to mark some points on a map of a game that has two-dimensional coordinates. We might expect to find a
function like the following:

.. code-block:: python

    def mark_coordinate(grid, coord):
        if 0 <= coord.x < grid.width and 0 <= coord.y < grid.height:
            grid[coord] = MARKED

Now, the part that checks the condition of the first if statement seems convoluted; it doesn't reveal the intention of
the code, it's not expressive, and worst of all it calls for code duplication (every part of the code where we need to
check the boundaries before proceeding will have to repeat that if statement).

What if the map itself (called grid on the code) could answer this question? Even better, what if the map could delegate
this action to an even smaller (and hence more cohesive) object? Therefore, we can ask the map if it contains a
coordinate, and the map itself can have information about its limit, and ask this object the following:

.. code-block:: python

    class Boundaries:
        def __init__(self, width, height):
            self.width = width
            self.height = height

        def __contains__(self, coord):
            x, y = coord
            return 0 <= x < self.width and 0 <= y < self.height

    class Grid:
        def __init__(self, width, height):
            self.width = width
            self.height = height
            self.limits = Boundaries(width, height)

        def __contains__(self, coord):
            return coord in self.limits

This code alone is a much better implementation. First, it is doing a simple composition and it's using delegation to
solve the problem. Both objects are really cohesive, having the minimal possible logic; the methods are short, and the
logic speaks for itself: ``coord in self.limits`` is pretty much a declaration of the problem to solve, expressing the
intention of the code.

From the outside, we can also see the benefits. It's almost as if Python is solving the problem for us:

.. code-block:: python

    def mark_coordinate(grid, coord):
        if coord in grid:
            grid[coord] = MARKED
