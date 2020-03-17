2. Properties
*************

When the object needs to just hold values, we can use regular attributes. Sometimes, we might want to do some
computations based on the state of the object and the values of other attributes. Most of the time, properties are a
good choice for this.

Properties are to be used when we need to define access control to some attributes in an object, which is another point
where Python has its own way of doing things. In other programming languages (like Java), you would create access
methods (getters and setters), but idiomatic Python would use properties instead.

The properties provide a built-in descriptor type that knows how to link an attribute to a
set of methods. ``property`` takes four optional arguments: ``fget``, ``fset``, ``fdel``, and ``doc``. The
last one can be provided to define a ``docstring`` function that is linked to the attribute as if
it were a method. Here is an example of a ``Rectangle`` class that can be controlled either by
direct access to attributes that store two corner points or by using
the ``width`` and ``height`` properties:

.. code-block:: python

    class Rectangle:

        def __init__(self, x1, y1, x2, y2):
            self.x1, self.y1 = x1, y1
            self.x2, self.y2 = x2, y2

        def _width_get(self):
            return self.x2 - self.x1

        def _width_set(self, value):
            self.x2 = self.x1 + value

        def _height_get(self):
            return self.y2 - self.y1

        def _height_set(self, value):
            self.y2 = self.y1 + value

        width = property(
            _width_get, _width_set,
            doc="rectangle width measured from left"
        )
        height = property(
            _height_get, _height_set,
            doc="rectangle height measured from top"
        )

        def __repr__(self):
            return "{}({}, {}, {}, {})".format(
                self.__class__.__name__,
                self.x1, self.y1, self.x2, self.y2
            )

The following is an example of such defined properties in an interactive session:

.. code-block:: python

    >>> rectangle = Rectangle(10, 10, 25, 34)
    >>> rectangle.width, rectangle.height
    (15, 24)
    >>> rectangle.width = 100
    >>> rectangle
    Rectangle(10, 10, 110, 34)
    >>> rectangle.height = 100
    >>> rectangle
    Rectangle(10, 10, 110, 110)
    >>> help(Rectangle)
    Help on class Rectangle in module sample_module:
    class Rectangle(builtins.object)
    |   Methods defined here:
    |
    |   __init__(self, x1, y1, x2, y2)
    |       Initialize self. See help(type(self)) for accurate signature.
    |   __repr__(self)
    |       Return repr(self).
    |
    | --------------------------------------------------------
    |   Data descriptors defined here:
    |   (...)
    |
    |   height
    |       rectangle height measured from top
    |
    |   width
    |       rectangle width measured from left

The properties make it easier to write descriptors, but must be handled carefully when
using inheritance over classes. The attribute created is made on the fly using the methods of
the current class and will not use methods that are overridden in the derived classes.

For instance, the following example will fail to override the implementation of
the ``fget`` method of the parent's class (``Rectangle``) ``width`` property:

.. code-block:: python

    >>> class MetricRectangle(Rectangle):
    ...     def _width_get(self):
    ...         return "{} meters".format(self.x2 - self.x1)
    ...
    >>> Rectangle(0, 0, 100, 100).width
    100

In order to resolve this, the whole property simply needs to be overwritten in the derived
class:

.. code-block:: python

    >>> class MetricRectangle(Rectangle):
    ...     def _width_get(self):
    ...         return "{} meters".format(self.x2 - self.x1)
    ...     width = property(_width_get, Rectangle.width.fset)
    ...
    >>> MetricRectangle(0, 0, 100, 100).width
    '100 meters'

Unfortunately, the preceding code has some maintainability issues. It can be a source of
confusion if the developer decides to change the parent class, but forgets to update the
property call. This is why overriding only parts of the property behavior is not advised.
Instead of relying on the parent class's implementation, it is recommended that you rewrite
all the property methods in the derived classes if you need to change how they work. In
most cases, this is the only option, because usually the change to the
property ``setter`` behavior implies a change to the behavior of ``getter`` as well.

Because of this, the best syntax for creating properties is to use property as a decorator.
This will reduce the number of method signatures inside the class and make the code more
readable and maintainable:

.. code-block:: python

    class Rectangle:
        def __init__(self, x1, y1, x2, y2):
            self.x1, self.y1 = x1, y1
            self.x2, self.y2 = x2, y2

        @property
        def width(self):
            """rectangle width measured from left"""
            return self.x2 - self.x1

        @width.setter
        def width(self, value):
            self.x2 = self.x1 + value

        @property
        def height(self):
            """rectangle height measured from top"""
            return self.y2 - self.y1

        @height.setter
        def height(self, value):
            self.y2 = self.y1 + value

This approach is much more compact than having custom methods prefixed with ``get_`` or ``set_``. It's clear what is
expected because it's just email.

.. note::

    Don't write custom ``get_*`` and ``set_*`` methods for all attributes on your objects. Most of the time, leaving
    them as regular attributes is just enough. If you need to modify the logic for when an attribute is retrieved or
    modified, then use properties.

You might find that properties are a good way to achieve command and query separation. Command and query
separation state that a method of an object should either answer to something or do something, but not both. If a method
of an object is doing something and at the same time it returns a status answering a question of how that operation
went, then it's doing more than one thing, clearly violating the principle that functions should do one thing, and one
thing only.

Depending on the name of the method, this can create even more confusion, making it harder for readers to understand
what the actual intention of the code is. For example, if a method is called ``set_email``, and we use it as
``if self.set_email("a@j.com"): ...``, what is that code doing? Is it setting the email to a@j.com? Is it checking if the
email is already set to that value? Both (setting and then checking if the status is correct)?

With properties, we can avoid this kind of confusion. The ``@property`` decorator is the query that will answer to
something, and the ``@<property_name>.setter`` is the command that will do something.

Another piece of good advice derived from this example is as follows: don't do more than one thing on a method. If you
want to assign something and then check the value, break that down into two or more sentences.

.. note::

    Methods should do one thing only. If you have to run an action and then check for the status, so that in separate
    methods that are called by different statements.
