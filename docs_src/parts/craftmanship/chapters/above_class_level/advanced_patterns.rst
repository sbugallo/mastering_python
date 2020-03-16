5. Advanced attribute access patterns
*************************************

When many C++ and Java programmers first learn Python, they are surprised by Python's
lack of a ``private`` keyword. The nearest concept is **name mangling**. Every time an attribute
is prefixed by ``__`` , it is renamed by the interpreter on the fly:

.. code-block:: python

    class MyClass:
        __secret_value = 1

Accessing the ``__secret_value`` attribute by its initial name will raise
an ``AttributeError`` exception:

.. code-block:: python

    >>> instance_of = MyClass()
    >>> instance_of.__secret_value
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    AttributeError: 'MyClass' object has no attribute '__secret_value'
    >>> dir(MyClass)
    ['_MyClass__secret_value', '__class__', '__delattr__', '__dict__',
    '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__',
    '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__module__',
    '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__',
    '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
    >>> instance_of._MyClass__secret_value
    1

This feature is provided to avoid name collision under inheritance, as the attribute is
renamed with the class name as a prefix. It is not a real privacy lock, since the attribute can
be accessed through its composed name. This feature could be used to protect the access of
some attributes, but, in practice, **``__`` should never be used**. When an attribute is not public,
the convention to use is a ``_`` prefix. This does not invoke any name mangling algorithm, but
just documents the attribute as a private element of the class and is the prevailing style.

Other mechanisms are available in Python that nicely separate the public part of the class
together with the private code. The descriptors and properties are interesting features of
Python that allow us to cleanly provide this kind of separation.

5.1. Descriptors
++++++++++++++++

A descriptor lets you customize what should be done when you refer to an attribute of an
object.

Descriptors are the base of a complex attribute access in Python. They are used internally to
implement properties, methods, class methods, static methods, and the ``super`` type. They
are classes that define how attributes of another class can be accessed. In other words, a
class can delegate the management of an attribute to another one.

The descriptor classes are based on three special methods that form the **descriptor protocol**:

- ``__set__(self, obj, value)``: This is called whenever the attribute is set. In
  the following examples, I will refer to this as a ``setter``.
- ``__get__(self, obj, owner=None)``: This is called whenever the attribute is
  read (referred to as a ``getter``).
- ``__delete__(self, obj)``: This is called when ``del`` is invoked on the attribute.

A descriptor that implements ``__get__()`` and ``__set__()`` is called a data descriptor. If it
just implements ``__get__()``, then it is called a non-data descriptor.

Methods of this protocol are, in fact, called by the object's
special ``__getattribute__()`` method (do not confuse it with ``__getattr__()``, which has
a different purpose) on every attribute lookup. Whenever such a lookup is performed,
either by using a dotted notation in the form of ``instance.attribute``, or by using
the ``getattr(instance, 'attribute')`` function call, the ``__getattribute__()`` method
is implicitly invoked and it looks for an attribute in the following order:

1. It verifies whether the attribute is a data descriptor on the class object of the
   instance.
2. If not, it looks to see whether the attribute can be found in
   the ``__dict__`` lookup of the instance object.
3. Finally, it looks to see whether the attribute is a non-data descriptor on the class
   object of the instance.

In other words, data descriptors take precedence over ``__dict__`` lookup,
and ``__dict__`` lookup takes precedence over non-data descriptors.

To make it clearer, here is an example from the official Python documentation that shows
how descriptors work on real code:

.. code-block:: python

    class RevealAccess(object):
        """A data descriptor that sets and returns values
        normally and prints a message logging their access.
        """
        def __init__(self, initval=None, name='var'):
            self.val = initval
            self.name = name

        def __get__(self, obj, objtype):
            print('Retrieving', self.name)
            return self.val

        def __set__(self, obj, val):
            print('Updating', self.name)
            self.val = val

    class MyClass(object):
        x = RevealAccess(10, 'var "x"')
        y = 5

Here is an example of using it in the interactive session:

.. code-block:: python

    >>> m = MyClass()
    >>> m.x
    Retrieving var "x"
    10
    >>> m.x = 20
    Updating var "x"
    >>> m.x
    Retrieving var "x"
    20
    >>> m.y
    5

The preceding example clearly shows that, if a class has the data descriptor for the given
attribute, then the descriptor's ``__get__()`` method is called to return the value every time
the instance attribute is retrieved, and ``__set__()`` is called whenever a value is assigned to
such an attribute. Although the case for the descriptor's ``__del__`` method is not shown in
the preceding example, it should be obvious now: it is called whenever an instance attribute
is deleted with the ``del instance.attribute`` statement or the ``delattr(instance,
'attribute')`` call.

The difference between data and non-data descriptors is important for the reasons
highlighted at the beginning of the section. Python already uses the descriptor protocol to
bind class functions to instances as methods. They also power the mechanism behind
the ``classmethod`` and ``staticmethod`` decorators. This is because, in fact, the function
objects are non-data descriptors too:

.. code-block:: python

    >>> def function(): pass
    >>> hasattr(function, '__get__')
    True
    >>> hasattr(function, '__set__')
    False


This is also true for functions created with lambda expressions:

.. code-block:: python

    >>> hasattr(lambda: None, '__get__')
    True
    >>> hasattr(lambda: None, '__set__')
    False

So, without ``__dict__`` taking precedence over non-data descriptors, we would not be able
to dynamically override specific methods on already constructed instances at runtime.
Fortunately, thanks to how descriptors work in Python, it is possible; so, developers may
use a popular technique called monkey patching to change the way in which instances
work without the need for subclassing.

5.1.1. Real-life example: lazily evaluated attributes
-----------------------------------------------------

One example usage of descriptors may be to delay initialization of the class attribute to the
moment when it is accessed from the instance. This may be useful if the initialization of
such attributes depends on the global application context. The other case is when such
initialization is simply expensive, but it is not known whether it will be used anyway when
the class is imported. Such a descriptor could be implemented as follows:

.. code-block:: python

    class InitOnAccess:

        def __init__(self, klass, *args, **kwargs):
            self.klass = klass
            self.args = args
            self.kwargs = kwargs
            self._initialized = None

        def __get__(self, instance, owner):
            if self._initialized is None:
                print('initialized!')
                self._initialized = self.klass(*self.args,
                **self.kwargs)
            else:
                print('cached!')
            return self._initialized

Here is an example usage:

.. code-block:: python

    >>> class MyClass:
    ...     lazily_initialized = InitOnAccess(list, "argument")
    ...
    >>> m = MyClass()
    >>> m.lazily_initialized
    initialized!
    ['a', 'r', 'g', 'u', 'm', 'e', 'n', 't']
    >>> m.lazily_initialized
    cached!
    ['a', 'r', 'g', 'u', 'm', 'e', 'n', 't']

The official OpenGL Python library available on PyPI under the ``PyOpenGL`` name uses a
similar technique to implement a ``lazy_property`` object that is both a decorator and a data
descriptor:

.. code-block:: python

    class lazy_property(object):
        def __init__(self, function):
            self.fget = function

        def __get__(self, obj, cls):
            value = self.fget(obj)
            setattr(obj, self.fget.__name__, value)
            return value

Such an implementation is similar to using the ``property`` decorator, but
the function that is wrapped with it is executed only once and then the class attribute is
replaced with a value returned by that function property. That technique is often useful
when there's a need to fulfil the following two requirements at the same time:

- An object instance needs to be stored as a class attribute that is shared between
  its instances (to save resources)
- This object cannot be initialized at the time of import because its creation process
  depends on some global application state/context

In the case of applications written using OpenGL, you can encounter this kind of situation
very often. For example, the creation of shaders in OpenGL is expensive because it requires
a compilation of code written in OpenGL Shading Language (GLSL). It is reasonable to
create them only once, and, at the same time, include their definition in close proximity to
classes that require them. On the other hand, shader compilations cannot be performed
without OpenGL context initialization, so it is hard to define and compile them reliably in a
global module namespace at the time of import.

The following example shows the possible usage of the modified version of
PyOpenGL's ``lazy_property`` decorator (here, ``lazy_class_attribute``) in some
imaginary OpenGL-based application. The highlighted change to the
original ``lazy_property`` decorator was required in order to allow the attribute to be
shared between different class instances:

.. code-block:: python

    import OpenGL.GL as gl
    from OpenGL.GL import shaders


    class lazy_class_attribute(object):

        def __init__(self, function):
            self.fget = function

        def __get__(self, obj, cls):
            value = self.fget(obj or cls)
            # note: storing in class object not its instance
            #   no matter if its a class-level or
            #   instance-level access
            setattr(cls, self.fget.__name__, value)
            return value


    class ObjectUsingShaderProgram(object):
        # trivial pass-through vertex shader implementation
        VERTEX_CODE = """
            #version 330 core
            layout(location = 0) in vec4 vertexPosition;
            void main(){
                gl_Position = vertexPosition;
            }
        """
        # trivial fragment shader that results in everything
        # drawn with white color
        FRAGMENT_CODE = """
            #version 330 core
            out lowp vec4 out_color;
            void main(){
                out_color = vec4(1, 1, 1, 1);
            }
        """

        @lazy_class_attribute
        def shader_program(self):
            print("compiling!")
            return shaders.compileProgram(
                shaders.compileShader(
                    self.VERTEX_CODE, gl.GL_VERTEX_SHADER
                ),
                shaders.compileShader(
                    self.FRAGMENT_CODE, gl.GL_FRAGMENT_SHADER
                )
            )

Like every advanced Python syntax feature, this one should also be used with caution and
documented well in code. For inexperienced developers, the altered class behavior might
be very confusing and unexpected, because descriptors affect the very basic part of class
behavior. Because of that, it is very important to make sure that all your team members are
familiar with descriptors and understand this concept well if it plays an important role in
your project's code base.

5.2. Properties
+++++++++++++++

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

5.3. Slots
++++++++++

An interesting feature that is very rarely used by developers is slots. They allow you to set a
static attribute list for a given class with the ``__slots__`` attribute, and skip the creation of
the ``__dict__`` dictionary in each instance of the class. They were intended to save memory
space for classes with very few attributes, since ``__dict__`` is not created at every instance.

Besides this, they can help to design classes whose signature needs to be frozen. For
instance, if you need to restrict the dynamic features of the language over a class, defining
slots can help:

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

This feature should be used carefully. When a set of available attributes is limited
using ``__slots__``, it is much harder to add something to the object dynamically. Some
techniques, such as monkey patching, will not work with instances of classes that have slots
defined. Fortunately, the new attributes can be added to the derived classes if they do not
have their own slots defined:

.. code-block:: python

    >>> class Unfrozen(Frozen):
    ...     pass
    ...
    >>> unfrozen = Unfrozen()
    >>> unfrozen.icy = False
    >>> unfrozen.icy
    False
