4. MRO and accessing methods from superclasses
**********************************************

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

4.1. Old-style classes and super in Python 2
++++++++++++++++++++++++++++++++++++++++++++

``super()`` in Python 2 works almost exactly the same as in Python 3. The only difference in
its call signature is that the shorter, zero-argument form is not available, so at least one of
the expected arguments must always be provided.

Another important thing for programmers to note who want to write cross-version
compatible code is that super in Python 2 works only for new-style classes. The earlier
versions of Python did not have a common ancestor for all classes in the form of
an object type. The old behavior was left in every Python 2.x branch release for backward
compatibility, so, in those versions, if the class definition has no ancestor specified, it is
interpreted as an old-style class, and it cannot use ``super``:

.. code-block:: python

    class OldStyle1:
        pass

    class OldStyle2(OldStyle1):
        pass

The new-style class in Python 2 must explicitly inherit from the object type or other new-
style class:

.. code-block:: python

    class NewStyleClass(object):
        pass

    class NewStyleClassToo(NewStyleClass):
        pass

Python 3 no longer maintains the concept of old-style classes, so any class that does not
inherit from any other class implicitly inherits from ``object``. This means that explicitly
stating that a class inherits from object may seem redundant. Standard good practice is to
not include redundant code, but removing such redundancy in this case is a good approach
only for projects that no longer target any of the Python 2 versions. Code that aims for
cross-version compatibility of Python must always include ``object`` as an ancestor of base
classes, even if this is redundant in Python 3. Not doing so will result in such classes being
interpreted as old-style, and this will eventually lead to issues that are very hard to
diagnose.

4.2. Understanding Python's Method Resolution Order
+++++++++++++++++++++++++++++++++++++++++++++++++++

Python MRO is based on C3, the MRO built for the Dylan programming language
(`<http://opendylan.org>`_). The reference document, written by Michele Simionato, can be
found at `<http://www.python.org/download/releases/2.3/mro>`_ . It describes how C3 builds
the **linearization** of a class, also called **precedence**, which is an ordered list of the ancestors.
This list is used to seek an attribute. The C3 algorithm is described in more detail later in
this section.

The MRO change was made to resolve an issue introduced with the creation of a common
base type (that is, ``object`` type). Before the change to the C3 linearization method, if a class
had two ancestors, the order in which methods were resolved was quite
simple to compute and track only for simple cases that didn't use multiple inheritance
model in a cascading way.

Here is an example of code, which, under Python 2, would not use C3 as an MRO:

.. code-block:: python

    class Base1:
        pass

    class Base2:
        def method(self):
            print('Base2')

    class MyClass(Base1, Base2):
        pass

    >>> MyClass().method()
    Base2

When ``MyClass().method()`` is called, the interpreter looks for the method in ``MyClass``,
then ``Base1``, and then eventually finds it in ``Base2``:

.. figure:: ../../../../_static/images/mro_1.jpg
    :width: 40%
    :align: center

When we introduce some ``CommonBase`` class at the top of our class hierarchy
(both ``Base1`` and ``Base2`` will inherit from it), things will get more
complicated. As a result, the simple resolution order that behaves according to the **left-to-
right depth first** rule is getting back to the top through the ``Base1`` class before looking into
the ``Base2`` class. This algorithm results in a counterintuitive output. In some cases, the
method that is executed may not be the one that is the closest in the inheritance tree.

Such an algorithm is still available in Python 2 for old-style classes. Here is an example of
the old method resolution in Python 2 using old-style classes:

.. code-block:: python

    class CommonBase:
        def method(self):
        print('CommonBase')

    class Base1(CommonBase):
        pass

    class Base2(CommonBase):
        def method(self):
            print('Base2')

    class MyClass(Base1, Base2):
        pass

.. figure:: ../../../../_static/images/mro_2.jpg
    :width: 40%
    :align: center

The following transcript from the interactive session shows that ``Base2.method()`` will not
be called despite the ``Base2`` class being closer in the class hierarchy to ``MyClass``
than ``CommonBase``:

.. code-block:: python

    >>> MyClass().method()
    CommonBase

Such an inheritance scenario is extremely uncommon, so this is more a problem of theory
than practice. The standard library does not structure the inheritance hierarchies in this
way, and many developers think that it is bad practice. But, with the introduction
of object at the top of the types hierarchy, the multiple inheritance problem pops up on
the C side of the language, resulting in conflicts when doing subtyping. You should also
note that every class in Python 3 has now got the same common ancestor. Since making it
work properly with the existing MRO involved too much work, a new MRO was a simpler
and quicker solution.

So, the same example run under Python 3 gives a different result:

.. code-block:: python

    class CommonBase:
        def method(self):
            print('CommonBase')

    class Base1(CommonBase):
        pass

    class Base2(CommonBase):
        def method(self):
            print('Base2')

    class MyClass(Base1, Base2):
        pass

And here is the usage example showing that C3 serialization will pick the method of the
closest ancestor:

.. code-block:: python

    >>> MyClass().method()
    Base2

.. tip::

    Note that the preceding behavior cannot be replicated in Python 2 without
    the ``CommonBase`` class explicitly inheriting from ``object``. Reasons as to
    why it may be useful to specify ``object`` as a class ancestor in Python 3,
    even if this is redundant, were already mentioned.

The Python MRO is based on a recursive call over the base classes. To summarize the
Michele Simionato paper referenced at the beginning of this section, the C3 symbolic
notation applied to our example is as follows:

.. code-block:: python

    L[MyClass(Base1, Base2)] = MyClass + merge(L[Base1], L[Base2], Base1, Base2)

Here, ``L[MyClass]`` is the linearization of ``MyClass``, and ``merge`` is a specific algorithm that
merges several linearization results.

So, a synthetic description would be, as Simionato says:

    *"The linearization of C is the sum of C plus the merge of the linearizations of the parents and the list of the parents."*

The ``merge`` algorithm is responsible for removing the duplicates and preserving the correct
ordering. It is described in the paper like this (adapted to our example):

    *Take the head of the first list, that is, L[Base1][0]; if this head is not in the tail of any of the other lists, then add it to the linearization of MyClass and remove it from the lists in the merge, otherwise look at the head of the next list and take it, if it is a good head.*

    *Then, repeat the operation until all the classes are removed or it is impossible to find good heads. In this case, it is impossible to construct the merge, Python 2.3 will refuse to create the MyClass class and will raise an exception.*

The ``head`` is the first element of a list and the ``tail`` contains the rest of the elements. For
example, in (``Base1``, ``Base2``, ..., ``BaseN``), ``Base1`` is the ``head`` , and (``Base2``, ``...``,
``BaseN``) is the ``tail``.

In other words, C3 does a recursive depth lookup on each parent to get a sequence of lists.
Then, it computes a left-to-right rule to merge all lists with a hierarchy disambiguation,
when a class is involved in several lists.

So the result is as follows:

.. code-block:: python

    def L(klass):
        return [k.__name__ for k in klass.__mro__]

    >>> L(MyClass)
    ['MyClass', 'Base1', 'Base2', 'CommonBase', 'object']

.. tip::

    The ``__mro__`` attribute of a class (which is read-only) stores the result of
    the linearization computation. Computation is done when the class
    definition is loaded.

    You can also call ``MyClass.mro()`` to compute and get the result. This is
    another reason why classes in Python 2 should be taken with an extra
    case. While old-style classes in Python 2 have some defined order in
    which methods are resolved, they do not provide the ``__mro__`` attribute
    and the ``mro()`` method. So, despite the order of resolution, it is wrong to
    say that they have MRO. In most cases, whenever someone refers to MRO
    in Python, it means that they are referring to the C3 algorithm described
    in this section.

4.3. Super pitfalls
+++++++++++++++++++

Now, back to the ``super()`` call. If you deal with multiple inheritance hierarchy, it can
become problematic. This is mainly due to the initialization of classes. In Python, the
initialization methods (that is, the ``__init__()`` methods) of base classes are not implicitly
called in ancestor classes if ancestor classes override ``__init__()``. In such cases, you need
to call superclass methods explicitly, and this can sometimes lead to initialization problems.

4.3.1. Mixing super and explicit class calls
--------------------------------------------

In the following example, taken from James Knight's website
(`<http://fuhm.net/super-harmful>`_), a ``C`` class that calls initialization methods of its parent
classes using the ``super().__init__()`` method will make the call to
the ``B.__init__()`` class to be called twice:

.. code-block:: python

    class A:
        def __init__(self):
            print("A", end=" ")
            super().__init__()

    class B:
        def __init__(self):
            print("B", end=" ")
            super().__init__()

    class C(A, B):
        def __init__(self):
            print("C", end=" ")
            A.__init__(self)
            B.__init__(self)

Here is the output:

.. code-block:: python

    >>> print("MRO:", [x.__name__ for x in C.__mro__])
    MRO: ['C', 'A', 'B', 'object']
    >>> C()
    C A B B <__main__.C object at 0x0000000001217C50>

In the preceding transcript we see that initialization of class ``C`` invokes the ``B.__init__()``
method twice. To avoid such issues, super should be used in the whole class hierarchy.
The problem is that sometimes, a part of such complex hierarchy may be located in a third-
party code. Many other related pitfalls on the hierarchy calls introduced by multiple
inheritances can be found on James's page.

Unfortunately, you cannot be sure that external packages use ``super()`` in their code.
Whenever you need to subclass some third-party class, it is always a good approach to take
a look inside its code and the code of other classes in the MRO. This may be tedious, but, as
a bonus, you get some information about the quality of code provided by such a package
and more understanding of its code. You may learn something new that way.

4.3.2. Heterogeneous arguments
------------------------------

Another issue with ``super`` usage occurs if methods of classes within the class hierarchy use
inconsistent argument sets. How can a class call its base class an ``__init__()`` code if it
doesn't have the same signature? This leads to the following problem:

.. code-block:: python

    class CommonBase:
        def __init__(self):
            print('CommonBase')
            super().__init__()

    class Base1(CommonBase):
        def __init__(self):
            print('Base1')
            super().__init__()

    class Base2(CommonBase):
        def __init__(self, arg):
            print('base2')
            super().__init__()

    class MyClass(Base1 , Base2):
        def __init__(self, arg):
            print('my base')
            super().__init__(arg)

An attempt to create a ``MyClass`` instance will raise ``TypeError`` due to a mismatch of the
parent classes' ``__init__()`` signatures:

.. code-block:: python

    >>> MyClass(10)
    my base
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "<stdin>", line 4, in __init__
    TypeError: __init__() takes 1 positional argument but 2 were given

One solution would be to use arguments and keyword arguments packing
with ``*args`` and ``**kwargs`` magic so that all constructors pass along all the parameters,
even if they do not use them:

.. code-block:: python

    class CommonBase:
        def __init__(self, *args, **kwargs):
            print('CommonBase')
            super().__init__()
    class Base1(CommonBase):
        def __init__(self, *args, **kwargs):
            print('Base1')
            super().__init__(*args, **kwargs)

    class Base2(CommonBase):
        def __init__(self, *args, **kwargs):
            print('base2')
            super().__init__(*args, **kwargs)

    class MyClass(Base1 , Base2):
        def __init__(self, arg):
            print('my base')
            super().__init__(arg)

With this approach, the parent class signatures will always match:

.. code-block:: python

    >>> _ = MyClass(10)
    my base
    Base1
    base2
    CommonBase

This is an awful fix though, because it makes all constructors accept any kind of
parameters. It leads to weak code, since anything can be passed and gone through. Another
solution is to use the explicit ``__init__()`` calls of specific classes in ``MyClass``, but this
would lead to the first pitfall.

4.4. Best practices
+++++++++++++++++++

To avoid all the aforementioned problems, and until Python evolves in this field, we need
to take into consideration the following points:

- **Multiple inheritance should be avoided**: It can be replaced with some design patterns.
- **Super usage has to be consistent**: In a class
  hierarchy, super should be used everywhere or nowhere.
  Mixing super and classic calls is a confusing practice. People tend
  to avoid super to render their code more explicit.
- **Explicitly inherit from an object in Python 3 if you target Python 2 too**: Classes
  without any ancestor specified are recognized as old-style classes in Python 2.
  Mixing old-style classes with new-style classes should be avoided in Python 2.
- **Class hierarchy has to be looked over when a parent class method is called**: To
  avoid any problems, every time a parent class method is called, a quick glance at
  the MRO involved (with __mro__ ) is necessary.