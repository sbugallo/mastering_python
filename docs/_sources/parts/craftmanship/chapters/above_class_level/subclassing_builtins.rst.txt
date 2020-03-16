3. Subclassing built-in types
*****************************

Subclassing built-in types in Python is pretty straightforward. A built-in type,
called ``object`` is a common ancestor for all built-in types, as well as for all user-defined
classes that have no explicit parent class specified. Thanks to this, every time you need to
implement a class that behaves almost like one of the built-in types, the best practice is to
subtype it.

Now, we will look at the code for a class called ``distinctdict``, which uses this technique.
It will be a subclass of the usual Python ``dict`` type. This new class will behave, in most
ways, like an ordinary Python ``dict`` type. But, instead of allowing multiple keys with the
same value, when someone tries to add a new entry with an identical value, it raises
a ``ValueError`` subclass with a help message.

As already stated, the built-in dict type is an object subclass:

.. code-block:: python

    >>> isinstance(dict(), object)
    True
    >>> issubclass(dict, object)
    True

It means that we could easily define our own dictionary-based class as a direct subclass of
that type, as follows:

.. code-block:: python

    class distinctdict(dict):
        ...

The previous approach would be totally valid, as a subclassing of ``dict``, ``list``, and ``str``
types has been allowed since Python 2.2. But, usually the better approach is to subclass one
of the corresponding types from the ``collections`` module:

- ``collections.UserDict``
- ``collections.UserList``
- ``collections.UserString``

These classes are usually easier to work with, as the underlying regular ``dict``, ``list`` , and
``str`` objects are stored as data attributes of these classes.

The following is an example implementation of the ``distinctdict`` type that overrides part
of the ordinary dictionary protocol to ensure that it contains only unique values:

.. code-block:: python

    from collections import UserDict


    class DistinctError(ValueError):
        """Raised when duplicate value is added to a distinctdict."""

    class distinctdict(UserDict):
        """Dictionary that does not accept duplicate values."""

        def __setitem__(self, key, value):
            if value in self.values():
                if (
                    (key in self and self[key] != value) or
                    key not in self
                ):
                    raise DistinctError("This value already exists for different key")

            super().__setitem__(key, value)

The following is an example of using distinctdict in an interactive session:

.. code-block:: python

    >>> my = distinctdict()
    >>> my['key'] = 'value'
    >>> my['other_key'] = 'value'
    Traceback (most recent call last):
        File "<input>", line 1, in <module>
        File "<input>", line 10, in __setitem__
    DistinctError: This value already exists for different key
    >>> my['other_key'] = 'value2'
    >>> my
    {'key': 'value', 'other_key': 'value2'}
    >>> my.data
    {'key': 'value', 'other_key': 'value2'}

If you take a look at your existing code, you may find a lot of classes that partially
implement the protocols or functionalities of the built-in types. These classes could be faster
and cleaner if implemented as subtypes of these types. The ``list`` type, for instance,
manages the sequences of any type and you can use it every time your class works
internally with a sequence or collection.

The following is a simple example of the ``Folder`` class that subclasses the Python ``list``
type to represent and manipulate the contents of directories in a tree-like structure:

.. code-block:: python

    from collections import UserList


    class Folder(UserList):
        def __init__(self, name):
            self.name = name

        def dir(self, nesting=0):
            offset = " " * nesting
            print(f"{offset}{self.name}")

            for element in self:
                if hasattr(element, 'dir'):
                    element.dir(nesting + 1)
                else:
                    print(f"{offset} {element}")

Note that we have actually subclassed the ``UserList`` class from the collections module
and not the bare ``list`` type. It is possible to subclass bare built-in types, such as ``string``,
``dict`` or ``set`` , but it is advisable to use their user counterparts from the ``collections``
module instead because they make subclassing a bit easier.

The following is an example use of our ``Folder`` class in an interactive session:

.. code-block:: python

    >>> tree = Folder('project')
    >>> tree.append('README.md')
    >>> tree.dir()
    project/
    README.md
    >>> src = Folder('src')
    >>> src.append('script.py')
    >>> tree.append(src)
    >>> tree.dir()
    project/
    README.md
    src/
    script.py
    >>> tree.remove(src)
    >>> tree.dir()
    project/
    README.md

.. tip::

    When you are about to create a new class that acts like a sequence or a
    mapping, think about its features and look over the existing built-in types.
    The ``collections`` module extends basic lists of built-in types with many
    useful containers. You will often end up using one of them without
    needing to create your custom subclasses.