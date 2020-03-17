2. Dictionaries
***************

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

2.1. Implementation details
+++++++++++++++++++++++++++

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

2.2. Weaknesses and alternatives
++++++++++++++++++++++++++++++++

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
