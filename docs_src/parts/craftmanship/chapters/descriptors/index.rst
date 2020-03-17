Descriptors
===========

Descriptors are another distinctive feature of Python that takes object-oriented
programming to another level, and their potential allows users to build more powerful and
reusable abstractions. Most of the time, the full potential of descriptors is observed in
libraries or frameworks.

A descriptor lets you customize what should be done when you refer to an attribute of an
object.

Descriptors are the base of a complex attribute access in Python. They are used internally to
implement properties, methods, class methods, static methods, and the ``super`` type. They
are classes that define how attributes of another class can be accessed. In other words, a
class can delegate the management of an attribute to another one.

.. important::

    Like every advanced Python syntax feature, this one should also be used with caution and
    documented well in code. For inexperienced developers, the altered class behavior might
    be very confusing and unexpected, because descriptors affect the very basic part of class
    behavior. Because of that, it is very important to make sure that all your team members are
    familiar with descriptors and understand this concept well if it plays an important role in
    your project's code base.

.. include:: first_look.rst
.. include:: types.rst
.. include:: descriptors_in_action.rst
.. include:: analysis.rst