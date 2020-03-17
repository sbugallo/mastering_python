Python's protocols: dunder methods and attributes
=================================================

The Python data model specifies a lot of specially named methods that can be overridden in
your custom classes to provide them with additional syntax capabilities. You can recognize
these methods by their specific naming conventions that wrap the method name with
**double underscores**. Because of this, they are sometimes referred to as **dunder**. It is simply
a speech shorthand for double underscores.

The most common and obvious example of such dunder methods is ``__init__()``, which is
used for class instance initialization:

.. code-block:: python

    class CustomUserClass:
        def __init__(self, initiatization_argument):
            ...

These methods, either alone or when defined in specific combination, constitute the so-called
language protocols. If an object implements specific language protocols, it becomes
compatible with specific parts of the Python language syntax. The following is the table of
the most important protocols within the Python language:

=============================================== ============================================== ================================================================================================================================================================
Protocol name                                   Methods                                        Description
=============================================== ============================================== ================================================================================================================================================================
Callable protocol                               ``__call__()``                                 Allows objects to be called with the parentheses syntax: ``instance()``
Descriptor protocols                            ``__set__()``, ``__get__()`` and ``__del__()`` Allows us to manipulate the attribute access pattern of classes
Container protocol                              ``__contains__()``                             Allows us to test whether or not an object contains some value using the ``in`` keyword: ``value in instance``
Iterable protocol                               ``__iter__()``                                 Allows objects to be iterated over using the ``for`` keyword: ``for value in instance``
Sequence protocols                              ``__len__()`` and ``__getitem__()``            Allows objects to be indexed with square bracket syntax and queried for length using a built-in function: ``item = instance[index]``, ``length = len(instance)``
=============================================== ============================================== ================================================================================================================================================================

These are the most important language protocols from the perspective of this chapter. The
full list is, of course, a lot longer. For instance, Python provides over 50 dunder methods
that allow us to emulate numeric values. Each of these methods is correlated to some
specific mathematical operator, and so could be considered a separate language protocol.
The full list of all the dunder methods can be found in the official documentation of the
Python data model (see `<https://docs.python.org/3/reference/datamodel.html>`_).

Language protocols are the foundation of the concept of interfaces in Python. One
implementation of Python interfaces is in abstract base classes that allow us to define an
arbitrary set of attributes and methods as an interface definition. These definitions of
interfaces in the form of abstract classes can be later used to test whether or not the given
object is compatible with a specific interface. The ``collections.abc`` module from the
Python standard library provides a collection of abstract base classes that refer to the most
common Python language protocol.

The same dunder convention is also used for specific attributes of custom user functions
and is used to store various metadata about Python objects. These attributes are as follows:

- ``__doc__``: A writable attribute that holds the function's documentation. It is, by
  default, populated by the ``docstring`` function.
- ``__name__``: A writable attribute that holds the function's name.
- ``__qualname__``: A writable attribute that holds the function's **qualified name**.
  The qualified name is a full dotted path to the object (with class names) in the
  global scope of the module where the object is defined.
- ``__module__``: A writable attribute that holds the name of the module that
  function belongs to.
- ``__defaults__``: A writable attribute that holds the default argument values if the
  function has any.
- ``__code__``: A writable attribute that holds the function's compile code object.
- ``__globals__``: A read-only attribute that holds the reference to the dictionary of
  global variables for that function's scope. The global scope for a function is the
  namespace of the module where this function is defined.
- ``__dict__``: A writable attribute that holds a dictionary of function attributes.
  Functions in Python are first-class objects, so they can have any arbitrary
  arguments defined, just like any other object.
- ``__closure__``: A read-only attribute that holds a tuple of cells with the function's
  free variables. Closure cells allow you to create parametrized function decorators.
- ``__annotations__``: A writable attribute that holds the function's argument and
  return annotations.
- ``__kwdefaults__``: A writable attribute that holds the default argument values
  for keyword-only arguments if the function has any.
