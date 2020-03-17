3. Sets
*******

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

3.1. Implementation details
+++++++++++++++++++++++++++

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
