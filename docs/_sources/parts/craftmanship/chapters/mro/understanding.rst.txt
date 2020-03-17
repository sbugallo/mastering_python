2. Understanding Python's Method Resolution Order
*************************************************

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
