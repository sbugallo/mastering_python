8. Caveats in Python
********************

8.1. Mutable default arguments
++++++++++++++++++++++++++++++

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

8.2. Extending built-in types
+++++++++++++++++++++++++++++

The correct way of extending built-in types such as lists, strings, and dictionaries is by means of the collections
module.

If you create a class that directly extends dict, for example, you will obtain results that are probably not what you
are expecting. The reason for this is that in CPython the methods of the class don't call each other (as they should),
so if you override one of them, this will not be reflected by the rest, resulting in unexpected outcomes. For example,
you might want to override ``__getitem__``, and then when you iterate the object with a for loop, you will notice that
the logic you have put on that method is not applied.

This is all solved by using collections.UserDict, for example, which provides a transparent interface to actual
dictionaries, and is more robust.

Let's say we want a list that was originally created from numbers to convert the values to strings, adding a prefix. The
first approach might look like it solves the problem, but it is erroneous:

.. code-block:: python

    class BadList(list):
         def __getitem__(self, index):
             value = super().__getitem__(index)
             if index % 2 == 0:
                prefix = "even"
             else:
                prefix = "odd"
             return f"[{prefix}] {value}"

At first sight, it looks like the object behaves as we want it to. But then, if we try to iterate it (after all, it is a
list), we find that we don't get what we wanted:

.. code-block:: python

    >>> bl = BadList((0, 1, 2, 3, 4, 5))
    >>> bl[0]
    '[even] 0'
    >>> bl[1]
    '[odd] 1'
    >>> "".join(bl)
    Traceback (most recent call last):
    ...
    TypeError: sequence item 0: expected str instance, int found

The join function will try to iterate (run a for loop over) the list, but expects values of type string. This should
work because it is exactly the type of change we made to the list, but apparently when the list is being iterated, our
changed version of the __getitem__ is not being called.

This issue is actually an implementation detail of CPython (a C optimization), and in other platforms such as PyPy it
doesn't happen. Regardless of this, we should write code that is portable and compatible in all implementations, so we
will fix it by extending not from list but from UserList:

.. code-block:: python

    from collections import UserList

    class GoodList(UserList):
        def __getitem__(self, index):
             value = super().__getitem__(index)
             if index % 2 == 0:
                prefix = "even"
             else:
                prefix = "odd"
             return f"[{prefix}] {value}"

And now things look much better:

.. code-block:: python

    >>> gl = GoodList((0, 1, 2))
    >>> gl[0]
    '[even] 0'
    >>> gl[1]
    '[odd] 1'
    >>> "; ".join(gl)
    '[even] 0; [odd] 1; [even] 2'

.. note::

    Don't extend directly from ``dict``, use ``collections.UserDict`` instead. For lists, use ``collections.UserList``, and
    for strings, use ``collections.UserString``.

