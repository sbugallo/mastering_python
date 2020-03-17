MRO and accessing methods from superclasses
===========================================

``super`` is a built-in class that can be used to access an attribute belonging to an object's
superclass.

.. important::

    The Python official documentation lists super as a built-in function, but,
    it's a built-in class, even if it is used like a function:

    .. code-block:: python

        >>> super
        <class 'super'>
        >>> isinstance(super, type)


Its usage is a bit confusing if you are used to accessing a class attribute or method by calling
the parent class directly and passing self as the first argument. This is a really old pattern,
but still can be found in some code bases (especially in legacy projects). See the following
code:

.. code-block:: python

    class Mama:
        def says(self):
            print('do your homework')

    class Sister(Mama):
        def says(self):
            Mama.says(self)
            print('and clean your bedroom')

Look particularly at the ``Mama.says(self)`` line. You can see here an explicit use of parent
class. This means that the ``says()`` method belonging to ``Mama`` will be called. But, the
instance on which it will be called is provided as the self argument, which is an instance
of ``Sister`` in this case.

Instead, the super usage would be as follows:

.. code-block:: python

    class Sister(Mama):
        def says(self):
            super(Sister, self).says()
            print('and clean your bedroom')

Alternatively, you can also use the shorter form of the ``super()`` call:

.. code-block:: python

    class Sister(Mama):
        def says(self):
            super().says()
            print('and clean your bedroom')

The shorter form of ``super`` (without passing any arguments) is allowed inside the methods,
but the usage of ``super`` is not limited to the body of methods. It can be used in any code
area where the explicit call to the method of superclass implementation is required. Still,
if super is not used inside the body of the method, then, all of its arguments are
mandatory:

.. code-block:: python

    >>> anita = Sister()
    >>> super(anita.__class__, anita).says()
    do your homework

The final and most important thing that should be noted about ``super`` is that its second
argument is optional. When only the first argument is provided, then super returns an
unbounded type. This is especially useful when working with ``classmethod``:

.. code-block:: python

    class Pizza:
        def __init__(self, toppings):
            self.toppings = toppings

        def __repr__(self):
            return "Pizza with " + " and ".join(self.toppings)

        @classmethod
        def recommend(cls):
            """Recommend some pizza with arbitrary toppings,"""
            return cls(['spam', 'ham', 'eggs'])

    class VikingPizza(Pizza):
        @classmethod
        def recommend(cls):
            """Use same recommendation as super but add extra spam"""
            recommended = super(VikingPizza).recommend()
            recommended.toppings += ['spam'] * 5
            return recommended

Note that the zero-argument ``super()`` form is also allowed for methods decorated with
the classmethod decorator. ``super()``, if called without arguments in such methods, is
treated as having only the first argument defined.

The use cases presented earlier are very simple to follow and understand, but when you
face a multiple inheritance schema, it becomes hard to use ``super``. Before explaining these
problems, you need to first understand when ``super`` should be avoided and how
the **Method Resolution Order (MRO)** works in Python.


.. include:: old_style.rst
.. include:: understanding.rst
.. include:: pitfalls.rst
.. include:: best_practices.rst
