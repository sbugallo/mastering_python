5. Container objects
********************

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
