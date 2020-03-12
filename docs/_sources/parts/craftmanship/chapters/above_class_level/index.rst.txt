Above the class level
=====================

In this chapter, we will focus on modern syntax elements of Python with regard to classes
and object-oriented programming. Here, we will perform an overview of the most advanced Python
syntax elements that will allow you to improve the code of your classes.

The Python class model as we know it evolved greatly during the history of Python 2. For a
long time, we lived in a world where two implementations of the object-oriented
programming paradigm coexisted in the same language. These two models were simply
referred to as old-style and new-style classes. Python 3 ended this dichotomy, so that only
the model known as the new-style class is available to developers. It is still important to
know how both of them worked in Python 2, because it will help you in porting old code
and writing backward compatible applications. Knowing how the object model changed
will also help you to understand why it is designed that way right now.

.. include:: protocols.rst
.. include:: data_classes.rst
.. include:: subclassing_builtins.rst
.. include:: mro.rst
.. include:: advanced_patterns.rst