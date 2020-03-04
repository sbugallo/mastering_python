Pythonic code
=============

Python provides a great set of data types. This is true for both numeric types and also
collections. Regarding the numeric types, there is nothing special about their syntax. There
are, of course, some differences for defining literals of every type and some (maybe) not
well-known details regarding operators, but there isn't a lot that could surprise you in
Python regarding the syntax for numeric types. Things change when it comes to collections
and strings. Despite the there should be only one way to do something mantra, the Python
developer is really left with plenty of choices. Some of the code patterns that seem intuitive
and simple to beginners are often considered non-Pythonic by experienced programmers,
because they are either inefficient or simply too verbose.

Such Pythonic patterns for solving common problems (many programmers call these
idioms) may often seem like only aesthetics. You couldn't be more wrong in thinking that.
Most of the idioms are driven by the fact that Python is implemented internally and how
the built-in structures and modules work. Knowing more about such details is essential for
a good understanding of the language. Unfortunately, the community itself is not free from
myths and stereotypes about how things in Python work. Only by digging deeper by
yourself will you be able to tell which of the popular statements about Python are really
true.

.. include:: strings_bytes.rst
.. include:: indexes_and_slices.rst
.. include:: context_managers.rst
.. include:: properties_attributes_and_methods.rst
.. include:: iterable_objects.rst
.. include:: container_objects.rst
.. include:: dynamic_attributes.rst
.. include:: callable_objects.rst
.. include:: docstrings.rst
.. include:: annotations.rst
.. include:: caveats.rst
