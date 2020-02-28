1. Docstrings
*************

Docstrings are basically documentation embedded in the source code. A **docstring** is basically a literal
string, placed somewhere in the code, with the intention of documenting that part of the logic. This
information it's meant to represent explanation, not justification.

Having comments in the code is a bad practice for multiple reasons. First, they represent our failure to
express our ideas in the code. Second, it can be misleading. Worst than having to spend some time reading a
complicated section is to read a comment on how it is supposed to work and figuring out that the code
actually does something different.

Sometimes, we cannot avoid having comments (maybe there is an error on a third-party library). In those cases,
placing a small but descriptive comment might be acceptable.

The reason why docstrings are a good thing to have in the code is that Python is dynamically typed. Python
will not enforce, nor check, anything like the value for any function's input parameters. Documenting the
expected input and output of a function is a good practice that will help the readers of that function
understand how it is supposed to work. This information is crucial for someone that hats to learn and
understand how a new code works, and how they can take advantage of it.

The docstring is not something separated or isolated from the code. It becomes part of the code, and you can
access it. When an object has a docstring defined, this becomes part of it via its ``__doc__`` attribute:

.. code-block:: python

    def sample():
        """Sample docstring"""
        return

    >>> sample.__doc__
    'Sample docstring'

There is, unfortunately, one downside to docstrings, and it is that, as it happens with all documentation, it
requires manual and constant maintenance. As the code changes, it will have to be updated. Another problem is
that for docstrings to be really useful, they have to be detailed, which requires multiple lines.