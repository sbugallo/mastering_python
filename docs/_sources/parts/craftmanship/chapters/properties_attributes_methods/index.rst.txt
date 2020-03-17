Properties, attributes and methods for objects
==============================================

All of the properties and functions of an object are public in Python, which is different from other languages where
properties can be public, private, or protected. That is, there is no point in preventing caller objects from invoking
any attributes an object has. This is another difference with respect to other programming languages in which you can
mark some attributes as private or protected.

There is no strict enforcement, but there are some conventions. An attribute that starts with an underscore is meant to
be private to that object, and we expect that no external agent calls it (but again, there is nothing preventing this).

.. include:: underscores.rst
.. include:: properties.rst
.. include:: slots.rst


