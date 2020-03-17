3. Slots
********

An interesting feature that is very rarely used by developers is slots. They allow you to set a
static attribute list for a given class with the ``__slots__`` attribute, and skip the creation of
the ``__dict__`` dictionary in each instance of the class. They were intended to save memory
space for classes with very few attributes, since ``__dict__`` is not created at every instance.

When a class defines the ``__slots__`` attribute, it can contain all the attributes that the class
expects and no more.

Trying to add extra attributes dynamically to a class that defines ``__slots__`` will result in
an ``AttributeError``. By defining this attribute, the class becomes static, so it will not have
a ``__dict__`` attribute where you can add more objects dynamically.

How, then, are its attributes retrieved if not from the dictionary of the object? By using
descriptors. Each name defined in a slot will have its own descriptor that will store the
value for retrieval later.

``__slots__`` can help to design classes whose signature needs to be frozen. For
instance, if you need to restrict the dynamic features of the language over a class, defining
slots can help:

.. code-block:: python

    class Coordinate2D:
        __slots__ = ("lat", "lon")
        def __init__(self, lat, lon):
            self.lat = lat
            self.lon = lon

        def __repr__(self):
            return f"{self.__class__.__name__}({self.lat}, {self.lon})"

While this is an interesting feature, it has to be used with caution because it is taking away
the dynamic nature of Python. In general, this ought to be reserved only for objects that we
know are static, and if we are absolutely sure we are not adding any attributes to them
dynamically in other parts of the code. Some
techniques, such as monkey patching, will not work with instances of classes that have slots
defined. Fortunately, the new attributes can be added to the derived classes if they do not
have their own slots defined:

.. code-block:: python

    >>> class Frozen:
    ...     __slots__ = ['ice', 'cream']
    ...
    >>> '__dict__' in dir(Frozen)
    False
    >>> 'ice' in dir(Frozen)
    True
    >>> frozen = Frozen()
    >>> frozen.ice = True
    >>> frozen.cream = None
    >>> frozen.icy = True
    Traceback (most recent call last): File "<input>", line 1, in <module>
    AttributeError: 'Frozen' object has no attribute 'icy'


    >>> class Unfrozen(Frozen):
    ...     pass
    ...
    >>> unfrozen = Unfrozen()
    >>> unfrozen.icy = False
    >>> unfrozen.icy
    False

As an upside of this, objects defined with slots use less memory, since they only need a
fixed set of fields to hold values and not an entire dictionary.