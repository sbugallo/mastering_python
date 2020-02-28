4. Analysis of descriptors
**************************

We have seen how descriptors work so far and explored some interesting situations in
which they contribute to clean design by simplifying their logic and leveraging more
compact classes.

Up to this point, we know that by using descriptors, we can achieve cleaner code,
abstracting away repeated logic and implementation details. But how do we know our
implementation of the descriptors is clean and correct? What makes a good descriptor? Are
we using this tool properly or over-engineering with it?

4.1. How Python uses descriptors internally
+++++++++++++++++++++++++++++++++++++++++++

Referring to the question as to what makes a good descriptor?, a simple answer would be
that a good descriptor is pretty much like any other good Python object. It is consistent with
Python itself. The idea that follows this premise is that analyzing how Python uses
descriptors will give us a good idea of good implementations so that we know what to
expect from the descriptors we write.

We will see the most common scenarios where Python itself uses descriptors to solve parts
of its internal logic, and we will also discover elegant descriptors and that they have been
there in plain sight all along.

4.1.1. Functions and methods
----------------------------

The most resonating case of an object that is a descriptor is probably a function. Functions
implement the ``__get__`` method, so they can work as methods when defined inside a class.
Methods are just functions that take an extra argument. By convention, the first argument
of a method is named "self", and it represents an instance of the class that the method is
being defined in. Then, whatever the method does with "self", would be the same as any
other function receiving the object and applying modifications to it.

In order words, when we define something like this:

.. code-block:: python

    class MyClass:
        def method(self, ...):
            self.x = 1

It is actually the same as if we define this:

.. code-block:: python

    class MyClass:
        pass

    def method(myclass_instance, ...):
        myclass_instance.x = 1
        method(MyClass())

So, it is just another function, modifying the object, only that it's defined inside the class,
and it is said to be bound to the object.

When we call something in the form of this:

.. code-block:: python

    instance = MyClass()
    instance.method(...)

Python is, in fact, doing something equivalent to this:

.. code-block:: python

    instance = MyClass()
    MyClass.method(instance, ...)

Notice that this is just a syntax conversion that is handled internally by Python. The way
this works is by means of descriptors.

Since functions implement the descriptor protocol (see the following listing) before calling
the method, the ``__get__()`` method is invoked first, and some transformations happen
before running the code on the internal callable:

.. code-block:: python

    >>> def function(): pass
    ...
    >>> function.__get__
    <method-wrapper '__get__' of function object at 0x...>

In the ``instance.method(...)`` statement, before processing all the arguments of the
callable inside the parenthesis, the "instance.method" part is evaluated.

Since ``method`` is an object defined as a class attribute, and it has a ``__get__`` method, this is
called. What this does is convert the function to a method, which means binding the
callable to the instance of the object it is going to work with.

Let's see this with an example so that we can get an idea of what Python might be doing
internally.

We will define a callable object inside a class that will act as a sort of function or method
that we want to define to be invoked externally. An instance of the ``Method`` class is
supposed to be a function or method to be used inside a different class. This function will
just print its three parameters: the instance that it received (which would be the
self parameter on the class it's being defined in), and two more arguments. Notice that in
the ``__call__()`` method, the self parameter does not represent the instance of
``MyClass``, but instead an instance of ``Method``. The parameter named instance is meant to
be a ``MyClass`` type of object:

.. code-block:: python

    class Method:
        def __init__(self, name):
            self.name = name

        def __call__(self, instance, arg1, arg2):
            print(f"{self.name}: {instance} called with {arg1} and {arg2}")

    class MyClass:
        method = Method("Internal call")

Under these considerations and, after creating the object, the following two calls should be
equivalent, based on the preceding definition:

.. code-block:: python

    instance = MyClass()
    Method("External call")(instance, "first", "second")
    instance.method("first", "second")

However, only the first one works as expected, as the second one gives an error:

.. code-block:: python

    Traceback (most recent call last):
    File "file", line, in <module>
    instance.method("first", "second")
    TypeError: __call__() missing 1 required positional argument: 'arg2'

We are seeing the same error we faced with a decorator. The arguments are being shifted to the left by one,
instance is taking the place of ``self``, ``arg1`` is going to be instance, and there is nothing to provide
for ``arg2``.

In order to fix this, we need to make ``Method`` a descriptor.

This way, when we call ``instance.method`` first, we are going to call its ``__get__()``, on
which we bind this callable to the object accordingly (bypassing the object as the first
parameter), and then proceed:

.. code-block:: python

    from types import MethodType

    class Method:
        def __init__(self, name):
            self.name = name

        def __call__(self, instance, arg1, arg2):
            print(f"{self.name}: {instance} called with {arg1} and {arg2}")

        def __get__(self, instance, owner):
            if instance is None:
                return self

            return MethodType(self, instance)

Now, both calls work as expected:

.. code-block:: python

    External call: <MyClass object at 0x...> called with fist and second
    Internal call: <MyClass object at 0x...> called with first and second

What we did is convert the function (actually the callable object we defined instead) to a
method by using ``MethodType`` from the ``types`` module. The first parameter of this class
should be a callable (``self``, in this case, is one by definition because it implements
``__call__``), and the second one is the object to bind this function to.

Something similar to this is what function objects use in Python so they can work as
methods when they are defined inside a class.

Since this is a very elegant solution, it's worth exploring it to keep it in mind as a Pythonic
approach when defining our own objects. For instance, if we were to define our own
callable, it would be a good idea to also make it a descriptor so that we can use it in classes
as class attributes as well.

4.1.2. Built-in decorators for methods
--------------------------------------

All ``@property``, ``@classmethod``, and ``@staticmethod`` decorators are descriptors.

We have mentioned several times that the idiom makes the descriptor return itself when it's
being called from a class directly. Since properties are actually descriptors, that is the
reason why, when we ask it from the class, we don't get the result of computing the
property, but the entire property object instead:

.. code-block:: python

    >>> class MyClass:
    ...     @property
    ...     def prop(self): pass
    ...
    >>> MyClass.prop
    <property object at 0x...>

For class methods, the ``__get__`` function in the descriptor will make sure that the class is
the first parameter to be passed to the function being decorated, regardless of whether it's
called from the class directly or from an instance. For static methods, it will make sure that
no parameters are bound other than those defined by the function, namely undoing the
binding done by ``__get__()`` on functions that make self the first parameter of that
function.

Let's take an example; we create a ``@classproperty`` decorator that works as the regular
``@property`` decorator, but for classes instead. With a decorator like this one, the following
code should be able to work:

.. code-block:: python

    class TableEvent:
        schema = "public"
        table = "user"

        @classproperty
        def topic(cls):
            prefix = read_prefix_from_config()
            return f"{prefix}{cls.schema}.{cls.table}"


    >>> TableEvent.topic
    'public.user'
    >>> TableEvent().topic
    'public.user'

4.1.3. Slots
------------

When a class defines the ``__slots__`` attribute, it can contain all the attributes that the class
expects and no more.

Trying to add extra attributes dynamically to a class that defines ``__slots__`` will result in
an ``AttributeError``. By defining this attribute, the class becomes static, so it will not have
a ``__dict__`` attribute where you can add more objects dynamically.

How, then, are its attributes retrieved if not from the dictionary of the object? By using
descriptors. Each name defined in a slot will have its own descriptor that will store the
value for retrieval later:

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
dynamically in other parts of the code.

As an upside of this, objects defined with slots use less memory, since they only need a
fixed set of fields to hold values and not an entire dictionary.

4.2. Implementing descriptors in decorators
+++++++++++++++++++++++++++++++++++++++++++

We now understand how Python uses descriptors in functions to make them work as
methods when they are defined inside a class. We have also seen examples of cases where
we can make decorators work by making them comply with the descriptor protocol by
using the ``__get__()`` method of the interface to adapt the decorator to the object it is being
called with. This solves the problem for our decorators in the same way that Python solves
the issue of functions as methods in objects.

The general recipe for adapting a decorator in such a way is to implement the ``__get__()``
method on it and use ``types.MethodType`` to convert the callable (the decorator itself) to a
method bound to the object it is receiving (the instance parameter received by ``__get__``).

For this to work, we will have to implement the decorator as an object, because otherwise, if
we are using a function, it will already have a ``__get__()`` method, which will be doing
something different that will not work unless we adapt it. The cleaner way to proceed is to
define a class for the decorator.

.. note:: Use a decorator class when defining a decorator that we want to apply to class methods, and implement the ``__get__()`` method on it.