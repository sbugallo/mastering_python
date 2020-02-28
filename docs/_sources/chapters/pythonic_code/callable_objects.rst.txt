8. Callable objects
*******************

It is possible (and often convenient) to define objects that can act as functions. One of the most common applications
for this is to create better decorators, but it's not limited to that.

The magic method ``__call__`` will be called when we try to execute our object as if it were a regular function. Every
argument passed to it will be passed along to the ``__call__`` method. The main advantage of implementing functions this way, through objects, is that objects have states, so we can save and
maintain information across calls.

When we have an object, a statement like this ``object(*args, **kwargs)`` is translated in Python to
``object.__call__(*args, **kwargs)``. This method is useful when we want to create callable objects that will work as
parametrized functions, or in some cases functions with memory.

The following listing uses this method to construct an object that when called with a parameter returns the number of
times it has been called with the very same value:

.. code-block:: python

    from collections import defaultdict

    class CallCount:
        def __init__(self):
            self._counts = defaultdict(int)

        def __call__(self, argument):
            self._counts[argument] += 1
            return self._counts[argument]


Some examples of this class in action are as follows:

.. code-block:: python

    >>> cc = CallCount()
    >>> cc(1)
    1
    >>> cc(2)
    1
    >>> cc(1)
    2
    >>> cc(1)
    3
    >>> cc("something")
    1