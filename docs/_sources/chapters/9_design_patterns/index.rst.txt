Common design patterns
======================

Design patterns have been a widespread topic in software engineering since their original
inception in the famous Gang of Four (GoF) book, Design Patterns: Elements of Reusable
Object-Oriented Software. Design patterns help to solve common problems with abstractions
that work for certain scenarios. When they are implemented properly, the general design of
the solution can benefit from them.
In this chapter we take a look at some of the most common design patterns, but not from
the perspective of tools to apply under certain conditions (once the patterns have been
devised), but rather we analyze how design patterns contribute to clean code. After
presenting a solution that implements a design pattern, we analyze how the final
implementation is comparatively better as if we had chosen a different path.
As part of this analysis, we will see how to concretely implement design patterns in Python.
As a result of that, we will see that the dynamic nature of Python implies some differences
of implementation, with respect to other static typed languages, for which many of the
design patterns were originally thought of. This means that there are some particularities
about design patterns that you should bear in mind when it comes to Python, and, in some
cases, trying to apply a design pattern where it doesn't really fit is non-Pythonic.

.. include:: considerations.rst
.. include:: design_patterns.rst
.. include:: null_object_pattern.rst
.. include:: final_thoughts.rst