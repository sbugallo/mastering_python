2. Decorators
*************

The decorator syntax was already explained, as a syntactic sugar for the following simple pattern:

.. code-block:: python

    def decorated_function():
        pass

    decorated_function = some_decorator(decorated_function)

This verbose form of function decoration clearly shows what the decorator does. It takes a
function object and modifies it at runtime. As a result, a new function (or anything else) is
created based on the previous function object with the same name. This decoration may be
a complex operation that performs some code introspection or decorated function to give
different results depending on how the original function was implemented. All this means
is that decorators can be considered as a metaprogramming tool.

This is good news. The basics of decorators are relatively easy to grasp and in most cases
make code shorter, easier to read, and also cheaper to maintain. Other metaprogramming
tools that are available in Python are more difficult to understand and master. Also, they
might not make the code simple at all.

2.1. Class decorators
+++++++++++++++++++++

One of the lesser known syntax features of Python are the class decorators. Their syntax
and implementation is exactly the same as function decorators. The only difference is that they are
expected to return a class instead of the function object. Here is an example class decorator
that modifies the ``__repr__()`` method to return the printable object representation, which
is shortened to some arbitrary number of characters:

.. code-block:: python

    def short_repr(cls):
        cls.__repr__ = lambda self: super(cls, self).__repr__()[:8]
        return cls

    @short_repr
    class ClassWithRelativelyLongName:
        pass

The following is what you will see in the output:

.. code-block:: python

    >>> ClassWithRelativelyLongName()
    <ClassWi

Of course, the preceding snippet is not an example of good code by any means. Still, it
shows how multiple language features that are explained in the previous chapter can be
used together, for example:

- Not only instances but also class objects can be modified at runtime
- Functions are descriptors too, so they can be added to the class at runtime because the actual method binding is performed on the attribute lookup as part of the descriptor protocol
- The ``super()`` call can be used outside of a class definition scope as long as proper arguments are provided
- Finally, class decorators can be used on class definitions

The other aspects of writing function decorators apply to the class decorators as well. Most
importantly, they can use closures and be parametrized. Taking advantage of these facts,
the previous example can be rewritten into the following more readable and maintainable
form:

.. code-block:: python

    def parametrized_short_repr(max_width=8):
        """Parametrized decorator that shortens representation"""

        def parametrized(cls):
            """Inner wrapper function that is actual decorator"""

            class ShortlyRepresented(cls):
                """Subclass that provides decorated behavior"""
                def __repr__(self):
                    return super().__repr__()[:max_width]

            return ShortlyRepresented

        return parametrized

The major drawback of using closures in class decorators this way is that the resulting
objects are no longer instances of the class that was decorated but instances of the subclass
that was created dynamically in the decorator function. Among others, this will affect the
class's ``__name__`` and ``__doc__`` attributes, as follows:

.. code-block:: python

    @parametrized_short_repr(10)
    class ClassWithLittleBitLongerLongName:
        pass

Such usage of class decorators will result in the following changes to the class metadata:

.. code-block:: python

    >>> ClassWithLittleBitLongerLongName().__class__
    <class 'ShortlyRepresented'>
    >>> ClassWithLittleBitLongerLongName().__doc__
    'Subclass that provides decorated behavior'

Unfortunately, this cannot be fixed as simply as we explained before. In class
decorators, you can't simply use the additional ``wraps`` decorator to preserve the original
class type and metadata. This makes use of the class decorators in this form limited in some
circumstances. They can, for instance, break results of automated documentation
generation tools.

Still, despite this single caveat, class decorators are a simple and lightweight alternative to
the popular mixin class pattern. Mixin in Python is a class that is not meant to be
instantiated, but is instead used to provide some reusable API or functionality to other
existing classes. Mixin classes are almost always added using multiple inheritance. Their
usage usually takes the following form:

.. code-block:: python

    class SomeConcreteClass(MixinClass, SomeBaseClass):
        pass

Mixins classes form a useful design pattern that is utilized in many libraries and
frameworks. To name one, Django is an example framework that uses them extensively.
While useful and popular, mixins can cause some trouble if not designed well, because, in
most cases, they require the developer to rely on multiple inheritance. As we stated earlier,
Python handles multiple inheritance relatively well, thanks to its clear MRO
implementation. Anyway, try to avoid subclassing multiple classes if you can. Multiple
inheritance makes code more complex and hard to reason about. This is why class
decorators may be a good replacement for mixin classes.
