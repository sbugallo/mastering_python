1. Mutable default arguments
****************************

Simply put, don't use mutable objects as the default arguments of functions. If you use mutable objects as default
arguments, you will get results that are not the expected ones. Consider the following erroneous function definition:

.. code-block:: python

    def wrong_user_display(user_metadata: dict = {"name": "John", "age": 30}):
        name = user_metadata.pop("name")
        age = user_metadata.pop("age")

        return f"{name} ({age})"

This has two problems, actually. Besides the default mutable argument, the body of the function is mutating a mutable
object, hence creating a side effect. But the main problem is the default argument for ``user_medatada``.

This will actually only work the first time it is called without arguments. For the second time, we call it without
explicitly passing something to ``user_metadata``. It will fail with a KeyError, like so:

.. code-block:: python

    >>> wrong_user_display()
    'John (30)'
    >>> wrong_user_display({"name": "Jane", "age": 25})
    'Jane (25)'
    >>> wrong_user_display()
    Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
     File ... in wrong_user_display
     name = user_metadata.pop("name")
    KeyError: 'name'

The explanation is simple—by assigning the dictionary with the default data to ``user_metadata`` on the definition of
the function, this dictionary is actually created once and the variable ``user_metadata`` points to it. The body of the
function modifies this object, which remains alive in memory so long as the program is running. When we pass a value to
it, this will take the place of the default argument we just created. When we don't want this object it is called again,
and it has been modified since the previous run; the next time we run it, will not contain the keys since they were
removed on the previous call.

The fix is also simple: we need to use None as a default sentinel value and assign the default on the body of the
function. Because each function has its own scope and life cycle, ``user_metadata`` will be assigned to the dictionary
every time None appears:

.. code-block:: python

    def user_display(user_metadata: dict = None):
        user_metadata = user_metadata or {"name": "John", "age": 30}
        name = user_metadata.pop("name")
        age = user_metadata.pop("age")

        return f"{name} ({age})"

