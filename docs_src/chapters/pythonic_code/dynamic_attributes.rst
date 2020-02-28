7. Dynamic attributes for objects
*********************************

It is possible to control the way attributes are obtained from objects by means of the ``__getattr__`` magic method.
When we call something like ``<myobject>.<myattribute>``, Python will look for ``<myattribute>`` in the dictionary of
the object, calling ``__getattribute__`` on it. If this is not found (namely, the object does not have the attribute we
are looking for), then the extra method, ``__getattr__``, is called, passing the name of the attribute (myattribute) as
a parameter. By receiving this value, we can control the way things should be returned to our objects. We can even
create new attributes, and so on.

In the following listing, the ``__getattr__`` method is demonstrated:

.. code-block:: python

    class DynamicAttributes:
        def __init__(self, attribute):
            self.attribute = attribute

        def __getattr__(self, attr):
            if attr.startswith("fallback_"):
                name = attr.replace("fallback_", "")
                return f"[fallback resolved] {name}"
            raise AttributeError(f"{self.__class__.__name__} has no attribute {attr}")

Here are some calls to an object of this class:

.. code-block::python

    >>> dyn = DynamicAttributes("value")
    >>> dyn.attribute
    'value'
    >>> dyn.fallback_test
    '[fallback resolved] test'
    >>> dyn.__dict__["fallback_new"] = "new value"
    >>> dyn.fallback_new
    'new value'
    >>> getattr(dyn, "something", "default")
    'default'

The first call is straightforward, we just request an attribute that the object has and get its value as a result. The
second is where this method takes action because the object does not have anything called ``fallback_test``, so the
``__getattr__`` will run with that value. Inside that method, we placed the code that returns a string, and what we get
is the result of that transformation.

The third example is interesting because there a new attribute named fallback_new is created (actually, this call would
be the same as running ``dyn.fallback_new = "new value"``), so when we request that attribute, notice that the logic we
put in ``__getattr__`` does not apply, simply because that code is never called.

Now, the last example is the most interesting one. There is a subtle detail here that makes a huge difference. Take
another look at the code in the ``__getattr__`` method. Notice the exception it raises when the value is not retrievable
AttributeError. This is not only for consistency (as well as the message in the exception) but also required by the
builtin ``getattr()`` function. Had this exception been any other, it would raise, and the default value would not be
returned.

.. note::

    Be careful when implementing a method so dynamic as ``__getattr__``, and use it with caution. When implementing it,
    raise AttributeError.
