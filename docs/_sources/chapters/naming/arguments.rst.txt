4. Best practices for arguments
*******************************

The signatures of functions and methods are the guardians of code integrity. They drive its
usage and build its APIs. Besides the naming rules that we have discussed previously,
special care has to be taken for arguments. This can be done through the following three
simple rules:

- Build arguments by iterative design.
- Trust the arguments and your tests.
- Use ``*args`` and ``**kwargs`` magic arguments carefully.

4.1. Building arguments by iterative design
+++++++++++++++++++++++++++++++++++++++++++

Having a fixed and well-defined list of arguments for each function makes the code more
robust. But this can't be done in the first version, so arguments have to be built by iterative
design. They should reflect the precise use cases the element was created for, and evolve
accordingly.

Consider the following example of the first versions of some ``Service`` class:

.. code-block:: python

    class Service: # version 1

        def _query(self, query, type):
            print('done')

        def execute(self, query):
            self._query(query, 'EXECUTE')

If you want to extend the signature of the ``execute()`` method with new arguments in a
way that preserves backward compatibility, you should provide default values for these
arguments as follows:

.. code-block:: python

    class Service(object): # version 2

        def _query(self, query, type, logger):
            logger('done')

        def execute(self, query, logger=logging.info):
            self._query(query, 'EXECUTE', logger)

The following example from an interactive session presents two styles of calling
the ``execute()`` method of the updated ``Service`` class:

.. code-block:: python

    >>> Service().execute('my query')
    # old-style call
    >>> Service().execute('my query', logging.warning)
    WARNING:root:done

4.2. Trusting the arguments and your tests
++++++++++++++++++++++++++++++++++++++++++

Given the dynamic typing nature of Python, some developers use assertions at the top of
their functions and methods to make sure the arguments have proper content, for example:

.. code-block:: python

    def divide(dividend, divisor):
        assert isinstance(dividend, (int, float))
        assert isinstance(divisor, (int, float))
        return dividend / divisor

This is often done by developers who are used to static typing and feel that something is
missing in Python.

This way of checking arguments is a part of the **Design by Contract (DbC)** programming
style, where preconditions are checked before the code is actually run.

The two main problems in this approach are as follows:

- DbC's code explains how it should be used, making it less readable
- This can make it slower, since the assertions are made on each call

The latter can be avoided with the ``-O`` option of the Python interpreter. In that case, all
assertions are removed from the code before the byte code is created, so that the checking is
lost.

In any case, assertions have to be done carefully, and should not be used to bend Python to
a statically typed language. The only use case for this is to protect the code from being
called nonsensically. If you really want to have some kind of static typing in Python, you
should definitely try MyPy or a similar static type checker that does not affect your code
runtime and allows you to provide type definitions in a more readable form as function
and variable annotations.

4.3. Using \*args and \*\*kwargs magic arguments carefully
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

The ``*args`` and ``**kwargs`` arguments can break the robustness of a function or method.
They make the signature fuzzy, and the code often starts to become a small argument
parser where it should not, for example:

.. code-block:: python

    def fuzzy_thing(**kwargs):
        if 'do_this' in kwargs:
            print('ok i did this')

        if 'do_that' in kwargs:
            print('that is done')

        print('ok')

    >>> fuzzy_thing(do_this=1)
    ok i did this
    ok
    >>> fuzzy_thing(do_that=1)
    that is done
    ok
    >>> fuzzy_thing(what_about_that=1)
    ok

If the argument list gets long and complex, it is tempting to add magic arguments. But this
is more a sign of a weak function or method that should be broken into pieces or refactored.

When ``*args`` is used to deal with a sequence of elements that are treated the same way in
the function, asking for a unique container argument such as an iterator is better, for
example:

.. code-block:: python

    def sum(*args): # okay
        total = 0
        for arg in args:
            total += arg

        return total


    def sum(sequence): # better!
        total = 0
        for arg in sequence:
            total += arg

        return total

For ``**kwargs``, the same rule applies. It is better to fix the named arguments to make the
method's signature meaningful, for example:

.. code-block:: python

    def make_sentence(**kwargs):
        noun = kwargs.get('noun', 'Bill')
        verb = kwargs.get('verb', 'is')
        adjective = kwargs.get('adjective', 'happy')
        return f'{noun} {verb} {adjective}'

    def make_sentence(noun='Bill', verb='is', adjective='happy'):
        return f'{noun} {verb} {adjective}'

Another interesting approach is to create a container class that groups several related
arguments to provide an execution context. This structure differs
from ``*args`` or ``**kwargs`` because it can provide internals that work over the values, and
can evolve independently. The code that uses it as an argument will not have to deal with
its internals.

For instance, a web request passed on to a function is often represented by an instance of a
class. This class is in charge of holding the data passed by the web server, as shown in the
following code:

.. code-block:: python

    def log_request(request): # version 1
        print(request.get('HTTP_REFERER', 'No referer'))

    def log_request(request): # version 2
        print(request.get('HTTP_REFERER', 'No referer'))
        print(request.get('HTTP_HOST', 'No host'))

Magic arguments cannot be avoided sometimes, especially in metaprogramming. For
instance, they are indispensable in the creation of decorators that work on functions with
any kind of signature.