4. Using __new__() for overriding instantiation
***********************************************

The special method ``__new__()`` is a static method that's responsible for creating class
instances. It is special-cased, so there is no need to declare it as static using
the ``staticmethod`` decorator. This ``__new__(cls, [,...])`` method is called prior to
the ``__init__()`` initialization method. Typically, the implementation of
overridden ``__new__()`` invokes its superclass version using ``super().__new__()`` with
suitable arguments and modifies the instance before returning it.

The following is an example class with the overridden ``__new__()`` method implementation
in order to count the number of class instances:

.. code-block:: python

    class InstanceCountingClass:

        instances_created = 0

        def __new__(cls, *args, **kwargs):
            print('__new__() called with:', cls, args, kwargs)
            instance = super().__new__(cls)
            instance.number = cls.instances_created
            cls.instances_created += 1

            return instance

        def __init__(self, attribute):
            print('__init__() called with:', self, attribute)
            self.attribute = attribute

Here is the log of the example interactive session that shows how
our InstanceCountingClass implementation works:

.. code-block:: python

    >>> from instance_counting import InstanceCountingClass
    >>> instance1 = InstanceCountingClass('abc')
    __new__() called with: <class '__main__.InstanceCountingClass'> ('abc',) {}
    __init__() called with: <__main__.InstanceCountingClass object at
    0x101259e10> abc
    >>> instance2 = InstanceCountingClass('xyz')
    __new__() called with: <class '__main__.InstanceCountingClass'> ('xyz',) {}
    __init__() called with: <__main__.InstanceCountingClass object at
    0x101259dd8> xyz
    >>> instance1.number, instance1.instances_created
    (0, 2)
    >>> instance2.number, instance2.instances_created
    (1, 2)

The ``__new__()`` method should usually return an instance of the featured class, but it is
also possible for it to return other class instances. If this does happen (a different class
instance is returned), then the call to the ``__init__()`` method is skipped. This fact is useful
when there is a need to modify creation/initialization behavior of immutable class instances
like some of Python's built-in types, as shown in the following code:

.. code-block:: python

    class NonZero(int):
        def __new__(cls, value):
            return super().__new__(cls, value) if value != 0 else None

        def __init__(self, skipped_value):
            # implementation of __init__ could be skipped in this case
            # but it is left to present how it may be not called
            print("__init__() called")
            super().__init__()

Let's review these in the following interactive session:

.. code-block:: python

    >>> type(NonZero(-12))
    __init__() called
    <class '__main__.NonZero'>
    >>> type(NonZero(0))
    <class 'NoneType'>
    >>> NonZero(-3.123)
    __init__() called
    -3

So, when should we use ``__new__()``? The answer is simple: only when ``__init__()`` is not
enough. One such case was already mentioned, that is, subclassing immutable built-in
Python types such as ``int``, ``str``, ``float``, ``frozenset``, and so on. This is because there was no
way to modify such an immutable object instance in the ``__init__()`` method once it was
created.

Some programmers can argue that ``__new__()`` may be useful for performing important
object initialization that may be missed if the user forgets to use
the ``super().__init__()`` call in the overridden initialization method. While it sounds
reasonable, this has a major drawback. With such an approach, it becomes harder for the
programmer to explicitly skip previous initialization steps if this is the already desired
behavior. It also breaks an unspoken rule of all initializations performed in ``__init__()``.

Because ``__new__()`` is not constrained to return the same class instance, it can be easily
abused. Irresponsible usage of this method might do a lot of harm to code readability, so it
should always be used carefully and backed with extensive documentation. Generally, it is
better to search for other solutions that may be available for the given problem, instead of
affecting object creation in a way that will break a basic programmers' expectations. Even
overridden initialization of immutable types can be replaced with more predictable and
well-established design patterns like the Factory Method.

There is at least one aspect of Python programming where extensive usage of
the ``__new__()`` method is well justified. These are metaclasses.
