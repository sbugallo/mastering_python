5. Iterable objects
*******************
In Python, we have objects that can be iterated by default: lists, tuples, sets and dictionaries. However, the built-in
iterable objects are not the only kind that we can have in a for loop. We could also create our own iterable, with the
logic we define for iteration.

In order to achieve this, we rely on magic methods. Iteration works in Python by its own protocol (namely the iteration
protocol). When you try to iterate an object in the form ``for e in myobject:``..., what Python checks at a very high
level are the following two things, in order:

- If the object contains one of the iterator methods ``__next__`` or ``__iter__``
- If the object is a sequence and has ``__len__`` and ``__getitem__``

Therefore, as a fallback mechanism, sequences can be iterated, and so there are two ways of customizing our objects to
be able to work on for loops.

5.1. Creating iterable objects
++++++++++++++++++++++++++++++

When we try to iterate an object, Python will call the ``iter()`` function over it. One of the first things this
function checks for is the presence of the ``__iter__`` method on that object, which, if present, will be executed.
``__iter__`` should return the iterator itself.

The following code creates an object that allows iterating over a range of dates, producing one day at a time on every
round of the loop:

.. code-block:: python

    from datetime import timedelta

    class DateRangeIterable:
        """An iterable that contains its own iterator object."""
        def __init__(self, start_date, end_date):
            self.start_date = start_date
            self.end_date = end_date
            self._present_day = start_date

     def __iter__(self):
        return self

     def __next__(self):
        if self._present_day >= self.end_date:
            raise StopIteration

        today = self._present_day
        self._present_day += timedelta(days=1)
        return today

This object is designed to be created with a pair of dates, and when iterated, it will produce each day in the interval
of specified dates, which is shown in the following code:

.. code-block:: python

    >>> for day in DateRangeIterable(date(2018, 1, 1), date(2018, 1, 5)):
    ...     print(day)

    2018-01-01
    2018-01-02
    2018-01-03
    2018-01-04

Here, the for loop is starting a new iteration over our object. At this point, Python will call the ``iter()`` function
on it, which in turn will call the ``__iter__`` magic method. On this method, it is defined to return ``self``,
indicating that the object is an iterable itself, so at that point every step of the loop will call the ``next()``
function on that object, which delegates to the ``__next__`` method. In this method, we decide how to produce the
elements and return one at a time. When there is nothing else to produce, we have to signal this to Python by raising
the StopIteration exception.

This means that what is actually happening is similar to Python calling ``next()`` every time on our object until there
is a StopIteration exception, on which it knows it has to stop the for loop:

This example works, but it has a small problem—once exhausted, the iterable will continue to be empty, hence raising
StopIteration. This means that if we use this on two or more consecutive for loops, only the first one will work, while
the second one will be empty:

.. code-block:: python

    >>> r1 = DateRangeIterable(date(2018, 1, 1), date(2018, 1, 5))
    >>> ", ".join(map(str, r1))
    '2018-01-01, 2018-01-02, 2018-01-03, 2018-01-04'
    >>> max(r1)
    Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
    ValueError: max() arg is an empty sequence

This is because of the way the iteration protocol works: an iterable constructs an iterator, and this one is the one
being iterated over. In our example, ``__iter__`` just returned self, but we can make it create a new iterator every
time it is called. One way of fixing this would be to create new instances of DateRangeIterable, which is not a
terrible issue, but we can make ``__iter__`` use a generator (which are iterator objects), which is being created
every time:

.. code-block:: python

    class DateRangeContainerIterable:

        def __init__(self, start_date, end_date):
            self.start_date = start_date
            self.end_date = end_date

        def __iter__(self):
            current_day = self.start_date
            while current_day < self.end_date:
                yield current_day

            current_day += timedelta(days=1)

And this time, it works:

.. code-block:: python

    >>> r1 = DateRangeContainerIterable(date(2018, 1, 1), date(2018, 1, 5))
    >>> ", ".join(map(str, r1))
    '2018-01-01, 2018-01-02, 2018-01-03, 2018-01-04'
    >>> max(r1)
    datetime.date(2018, 1, 4)

The difference is that each for loop is calling ``__iter__`` again, and each one of those is creating the generator
again. This is called a container iterable.

..note:: In general, it is a good idea to work with container iterables when dealing with generators.

5.2. Creating sequences
+++++++++++++++++++++++

Maybe our object does not define the ``__iter__()`` method, but we still want to be able to iterate over it. If
``__iter__`` is not defined on the object, the ``iter()`` function will look for the presence of ``__getitem__``, and if
this is not found, it will raise TypeError.

A sequence is an object that implements ``__len__`` and ``__getitem__`` and expects to be able to get the elements it
contains, one at a time, in order, starting at zero as the first index. This means that you should be careful in the
logic so that you correctly implement ``__getitem__`` to expect this type of index, or the iteration will not work.

The example from the previous section had the advantage that it uses less memory. This means that is only holding one
date at a time, and knows how to produce the days one by one. However, it has the drawback that if we want to get the
n-th element, we have no way to do so but iterate n-times until we reach it. This is a typical trade-off in computer
science between memory and CPU usage.

The implementation with an iterable will use less memory, but it takes up to O(n) to get an element, whereas
implementing a sequence will use more memory (because we have to hold everything at once), but supports indexing in
constant time, O(1).

This is what the new implementation might look like:

.. code-block:: python

    class DateRangeSequence:
         def __init__(self, start_date, end_date):
             self.start_date = start_date
             self.end_date = end_date
             self._range = self._create_range()

         def _create_range(self):
             days = []
             current_day = self.start_date

             while current_day < self.end_date:
                 days.append(current_day)
                 current_day += timedelta(days=1)

             return days

         def __getitem__(self, day_no):
            return self._range[day_no]

         def __len__(self):
            return len(self._range)

Here is how the object behaves:

.. code-block:: python

    >>> s1 = DateRangeSequence(date(2018, 1, 1), date(2018, 1, 5))
    >>> for day in s1:
    ...     print(day)
    2018-01-01
    2018-01-02
    2018-01-03
    2018-01-04
    >>> s1[0]
    datetime.date(2018, 1, 1)
    >>> s1[3]
    datetime.date(2018, 1, 4)
    >>> s1[-1]
    datetime.date(2018, 1, 4)

In the preceding code, we can see that negative indices also work. This is because the DateRangeSequence object
delegates all of the operations to its wrapped object (a list), which is the best way to maintain compatibility and a
consistent behavior.

Evaluate the trade-off between memory and CPU usage when deciding which one of the two possible implementations to use.
In general, the iteration is preferable (and generators even more), but keep in mind the requirements of every case.