1. Old-style classes and super in Python 2
******************************************

``super()`` in Python 2 works almost exactly the same as in Python 3. The only difference in
its call signature is that the shorter, zero-argument form is not available, so at least one of
the expected arguments must always be provided.

Another important thing for programmers to note who want to write cross-version
compatible code is that super in Python 2 works only for new-style classes. The earlier
versions of Python did not have a common ancestor for all classes in the form of
an object type. The old behavior was left in every Python 2.x branch release for backward
compatibility, so, in those versions, if the class definition has no ancestor specified, it is
interpreted as an old-style class, and it cannot use ``super``:

.. code-block:: python

    class OldStyle1:
        pass

    class OldStyle2(OldStyle1):
        pass

The new-style class in Python 2 must explicitly inherit from the object type or other new-
style class:

.. code-block:: python

    class NewStyleClass(object):
        pass

    class NewStyleClassToo(NewStyleClass):
        pass

Python 3 no longer maintains the concept of old-style classes, so any class that does not
inherit from any other class implicitly inherits from ``object``. This means that explicitly
stating that a class inherits from object may seem redundant. Standard good practice is to
not include redundant code, but removing such redundancy in this case is a good approach
only for projects that no longer target any of the Python 2 versions. Code that aims for
cross-version compatibility of Python must always include ``object`` as an ancestor of base
classes, even if this is redundant in Python 3. Not doing so will result in such classes being
interpreted as old-style, and this will eventually lead to issues that are very hard to
diagnose.
