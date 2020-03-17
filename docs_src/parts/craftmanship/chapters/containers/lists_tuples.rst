1. Lists and tuples
*******************

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

1.1. Implementation details
+++++++++++++++++++++++++++

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

1.2. List comprehensions
++++++++++++++++++++++++

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

1.3. Other idioms
+++++++++++++++++

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
