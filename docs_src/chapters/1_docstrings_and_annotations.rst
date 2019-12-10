Docstrings and annotations
==========================

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
understand how it is supposed to work.

This information is crucial for someone that hats to learn and undestand how a new code works, and how they
can take