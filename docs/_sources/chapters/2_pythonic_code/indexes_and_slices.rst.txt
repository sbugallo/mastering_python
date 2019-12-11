1. Indexes and slices
*********************

In Python, some data structures or types support accsing it's elements by index. The first element is placed in the
index number zero. How would you access the last element of a list?:

.. code-block:: python

    >>> numbers = (1, 2, 3, 4, 5)
    >>> numbers[-1]
    5
    >>> numbers[-3]
    3

In addition, we can obtain many elements by using ``slice``:

.. code-block:: python

    >>> numbers[2:5]
    (3, 4, 5)

In this case, the syntax means that we get all of the elements on the tuple, starting from the index of the first number
(inclusive), up to the index on the second one (not including it).

You can exclude either one of the intervals, start or stop, and in that case, it will act from the beginning or end of
the sequence:

.. code-block:: python

    >>>  numbers[:3]
    (1, 2, 3)
    >>> numbers[3:]
    (4, 5)
    >>> numbers[::]
    (1, 2, 3, 4, 5)
    >>> numbers[1:5:2]
    (2, 4)

In the first example, it will everything up to index 3. In the second example, it will get all numbers starting from
index 3. In the third example, where both ends are excluded, it is actually creating a copy of the original tuple. The
last example includes a third parameter, which is the step.

In all of these cases, when we pass intervals to a sequence, what is actually happening is that we are passing a
``slice``. Note that it is a built-in object in Python that you can build yourself and pass directly:

.. code-block:: python

    >>> interval = slice(1,5,2)
    >>>  numbers[interval]
    (2, 4)
    >>> interval = slice(None, 3)
    >>>  numbers[:3] == numbers[interval]
    True

1.1. Creating your own sequences
++++++++++++++++++++++++++++++++

The functionality we just discussed works thanks to a magic method called ``__getitem__``. This is the method that is
called when something like ``object[key]`` is called, passing the key as a parameter. A sequence is an object that
implements both ``__getitem__`` and ``__len__``, and for this reason, it can be iterated over.

In the case that your class is a wrapper around a standard library object, you might as well delegate the behavior as
much as possible to the underlying object. This means that if your class is actually a wrapper on the list, call all of
the same methods on that list to make sure that it remains compatible. In the following listing, we can see an example
of how an object wraps a list, and for the methods we are interested in, we just delegate to its corresponding version
on the list object:

.. code-block:: python

    class Items:
         def __init__(self, *values):
            self._values = list(values)

         def __len__(self):
            return len(self._values)

         def __getitem__(self, item):
            return self._values.__getitem__(item)

If you are implementing your own sequence then keep in mind the following points:

- When indexing by a range, the result should be an instance of the same type of the class.
- In the range provided by the slice, respect the semantics that Python uses, excluding the element at the end.
