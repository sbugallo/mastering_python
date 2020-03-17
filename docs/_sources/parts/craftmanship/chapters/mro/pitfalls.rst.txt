3. Super pitfalls
*****************

Now, back to the ``super()`` call. If you deal with multiple inheritance hierarchy, it can
become problematic. This is mainly due to the initialization of classes. In Python, the
initialization methods (that is, the ``__init__()`` methods) of base classes are not implicitly
called in ancestor classes if ancestor classes override ``__init__()``. In such cases, you need
to call superclass methods explicitly, and this can sometimes lead to initialization problems.

3.1. Mixing super and explicit class calls
++++++++++++++++++++++++++++++++++++++++++

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

3.2. Heterogeneous arguments
++++++++++++++++++++++++++++

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
