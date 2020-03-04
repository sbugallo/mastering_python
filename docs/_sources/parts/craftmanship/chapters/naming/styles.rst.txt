2. Naming styles
****************

The different naming styles used in Python are:

- CamelCase
- mixedCase
- UPPERCASE and UPPER_CASE_WITH_UNDERSCORES
- lowercase and lower_case_with_underscores
- _leading and trailing\_ underscores, and sometimes __doubled__ underscores

Lowercase and uppercase elements are often a single word, and sometimes a few words
concatenated. With underscores, they are usually abbreviated phrases. Using a single word
is better. The leading and trailing underscores are used to mark the privacy and special
elements.

These styles are applied to the following:

- Variables
- Functions and methods
- Properties
- Classes
- Modules
- Packages

2.1. Variables
++++++++++++++

There are the following two kinds of variables in Python:

- **Constants**: These define values that are not supposed to change during program execution
- **Public and private variables**: These hold the state of applications that can change during program execution

2.1.1. Constants
----------------

For constant global variables, an uppercase with an underscore is used. It informs the
developer that the given variable represents a constant value.

.. note::

    There are no real constants in Python like those in C++, where const can
    be used. You can change the value of any variable. That's why Python
    uses a naming convention to mark a variable as a constant.

For example, the ``doctest`` module provides a list of option flags and directives
(`http://docs.python.org/lib/doctest-options.html <http://docs.python.org/lib/doctest-options.html>`_)
that are small sentences, clearly defining what each option is intended for, for example:

.. code-block:: python

    from doctest import IGNORE_EXCEPTION_DETAIL
    from doctest import REPORT_ONLY_FIRST_FAILURE

These variable names seem rather long, but it is important to clearly describe them. Their
usage is mostly located in the initialization code rather than in the body of the code itself, so
this verbosity is not annoying.

.. note::

    Abbreviated names obfuscate the code most of the time. Don't be afraid of
    using complete words when an abbreviation seems unclear.

Some constants' names are also driven by the underlying technology. For instance,
the ``os`` module uses some constants that are defined on the C side, such as
the ``EX_XXX`` series, that defines UNIX exit code numbers. Same name code can be found, as
in the following example, in the system's ``sysexits.h`` C headers files:

.. code-block:: python

    import os
    import sys


    sys.exit(os.EX_SOFTWARE)

Another good practice when using constants is to gather all of them at the top of a module
that uses them. It is also common to combine them under new variables if they are flags or
enumerations that allow for such operations, for example:

.. code-block:: python

    import doctest
    TEST_OPTIONS = (doctest.ELLIPSIS | doctest.NORMALIZE_WHITESPACE | doctest.REPORT_ONLY_FIRST_FAILURE)

2.1.1.1. Naming and usage
~~~~~~~~~~~~~~~~~~~~~~~~~

Constants are used to define a set of values the program relies on, such as the default
configuration filename.

A good practice is to gather all the constants in a single file in the package. That is how
Django works, for instance. A module named ``settings.py`` provides all the constants as
follows:


.. code-block:: python

    SQL_USER = 'tarek'
    SQL_PASSWORD = 'secret'
    SQL_URI = 'postgres://%s:%s@localhost/db' % (SQL_USER, SQL_PASSWORD)
    MAX_THREADS = 4

Another approach is to use a configuration file that can be parsed with
the ``ConfigParser`` module, or another configuration parsing tool. But some people argue
that it is rather an overkill to use another file format in a language such as Python, where a
source file can be edited and changed as easily as a text file.

For options that act like flags, a common practice is to combine them with Boolean
operations, as the ``doctest`` and ``re`` modules do. The pattern taken from ``doctest`` is quite
simple, as shown in the following code:

.. code-block:: python

    OPTIONS = {}

    def register_option(name):
        return OPTIONS.setdefault(name, 1 << len(OPTIONS))

    def has_option(options, name):
        return bool(options & name)

    # now defining options
    BLUE = register_option('BLUE')
    RED = register_option('RED')
    WHITE = register_option('WHITE')

This code allows for the following usage:

.. code-block:: python

    >>> # let's try them
    >>> SET = BLUE | RED
    >>> has_option(SET, BLUE)
    True
    >>> has_option(SET, WHITE)
    False

When you define a new set of constants, avoid using a common prefix for them, unless the
module has several independent sets of options. The module name itself is a common
prefix.

Another good solution for option-like constants would be to use the ``Enum`` class from the
built-in ``enum`` module and simply rely on the ``set`` collection instead of the binary operators.

.. important::

    Using binary bit-wise operations to combine options is common in
    Python. The inclusive OR ( ``|`` ) operator will let you combine several
    options in a single integer, and the AND ( ``&`` ) operator will let you check
    that the option is present in the integer (refer to
    the ``has_option`` function).

2.1.2. Public and private variables
-----------------------------------

For global variables that are mutable and freely available through imports, a lowercase
letter with an underscore should be used when they do not need to be protected. If a
variable shouldn't be used and modified outside of its origin module we consider it a
private member of that module. A leading underscore, in that case, can mark the variable as
a private element of the package, as shown in the following code:

.. code-block:: python

    _observers = []

    def add_observer(observer):
        _observers.append(observer)

    def get_observers():
        """Makes sure _observers cannot be modified."""
        return tuple(_observers)

Variables that are located in functions, and methods, follow the same rules as public
variables and are never marked as private since they are local to the function context.

For class or instance variables, you should use the private marker (the leading underscore)
if making the variable a part of the public signature does not bring any useful information,
or is redundant. In other words, if the variable is used only internally for the purpose of
some other method that provides an actual public feature, it is better to make it private.

For instance, the attributes that are powering a property are good private citizens, as shown
in the following code:

.. code-block:: python

    class Citizen(object):

        def __init__(self, first_name, last_name):
            self._first_name = first_name
            self._last_name = last_name

        @property
        def full_name(self):
            return f"{self._first_name} {self._last_name}"

Another example would be a variable that keeps some internal state that should not be
disclosed to other classes. This value is not useful for the rest of the code, but participates in
the behavior of the class:

.. code-block:: python

    class UnforgivingElephant(object):

        def __init__(self, name):
            self.name = name
            self._people_to_stomp_on = []

        def get_slapped_by(self, name):
            self._people_to_stomp_on.append(name)
            print('Ouch!')

        def revenge(self):
            print('10 years later...')
            for person in self._people_to_stomp_on:
                print('%s stomps on %s' % (self.name, person))

Here is what you'll see in an interactive session:

.. code-block:: python

    >>> joe = UnforgivingElephant('Joe')
    >>> joe.get_slapped_by('Tarek')
    Ouch!
    >>> joe.get_slapped_by('Bill')
    Ouch!
    >>> joe.revenge()
    10 years later...
    Joe stomps on Tarek
    Joe stomps on Bill

2.2. Functions and methods
++++++++++++++++++++++++++

Functions and methods should be in lowercase with underscores. This rule was not always
true in the old standard library modules. Python 3 did a lot of reorganization of the
standard library, so most of the functions and methods have a consistent letter case. Still,
for some modules such as ``threading``, you can access the old function names that
used ``mixedCase`` (for example, ``currentThread``). This was left to allow easier backward
compatibility, but if you don't need to run your code in older versions of Python, then you
should avoid using these old names.

This way of writing methods was common before the lowercase norm became the standard,
and some frameworks, such as Zope and Twisted, are also still using ``mixedCase`` for
methods. The community of developers working with them is still quite large. So the choice
between ``mixedCase`` and lowercase with an underscore is definitely driven by the libraries
you are using.

As a Zope developer, it is not easy to stay consistent because building an application that
mixes pure Python modules and modules that import Zope code is difficult. In Zope, some
classes mix both conventions because the code base is still evolving and Zope developers
try to adopt the common conventions accepted by so many.

A decent practice in this kind of library environment is to use ``mixedCase`` only for elements
that are exposed in the framework, and to keep the rest of the code in PEP 8 style.

It is also worth noting that developers of the Twisted project took a completely different
approach to this problem. The Twisted project, same as Zope, predates the PEP 8
document. It was started when there were no official guidelines for Python code style, so it
had its own guidelines. Stylistic rules about the indentation, docstrings, line lengths, and so
on could be easily adopted. On the other hand, updating all the code to match naming
conventions from PEP 8 would result in completely broken backward compatibility. And
doing that for such a large project as Twisted is infeasible. So Twisted adopted as much of
PEP 8 as possible and left things such as ``mixedCase`` for variables, functions, and methods
as part of its own coding standard. And this is completely compatible with the PEP 8
suggestion because it exactly says that consistency within a project is more important than
consistency with PEP 8's style guide.

2.2.1 The private controversy
-----------------------------

For private methods and functions, we usually use a single leading underscore. This is only
a naming convention and has no syntactical meaning. But it doesn't mean that leading
underscores have no syntactical meaning at all. When a method has two leading
underscores, it is renamed on the fly by the interpreter to prevent a name collision with a
method from any subclass. This feature of Python is called **name mangling**.

So some people tend to use a double leading underscore for their private attributes to avoid
name collision in the subclasses, for example:

.. code-block:: python

    class Base(object):
        def __secret(self):
            print("don't tell")
        def public(self):
            self.__secret()

    class Derived(Base):
        def __secret(self):
            print("never ever")

From this you will see the following output:

.. code-block:: python

    >>> Base.__secret
    Traceback (most recent call last):
        File "<input>", line 1, in <module>
    AttributeError: type object 'Base' has no attribute '__secret'
    >>> dir(Base)
    ['_Base__secret', ..., 'public']
    >>> Base().public()
    don't tell
    >>> Derived().public()
    don't tell

The original motivation for name mangling in Python was not to provide the same isolation
primitive as a private keyword in C++ but to make sure that some base classes implicitly
avoid collisions in subclasses, especially if they are intended to be used in multiple
inheritance contexts (for example, as mixin classes). But using it for every attribute that isn't
public obfuscates the code and makes it extremely hard to extend. This is not Pythonic at
all.

For more information on this topic, an interesting thread occurred in the Python-Dev
mailing list many years ago, where people argued on the utility of name mangling and its
fate in the language. It can be found at

`http://mail.python.org/pipermail/python-dev/2005-December/058555.html <http://mail.python.org/pipermail/python-dev/2005-December/058555.html>`_ .

2.2.2. Special methods
----------------------

Special methods (`https://docs.python.org/3/reference/datamodel.html#special-method-names <https://docs.python.org/3/reference/datamodel.html#special-method-names>`_)
start and end with a double underscore and form so-called protocols of the
language. Some developers
used to call them *dunder* methods as a portmanteau of double underscore. They are used for
operator overloading, container definitions, and so on. For the sake of readability, they
should be gathered at the beginning of class definitions, as shown in the following code:

.. code-block:: python

    class WeirdInt(int):

        def __add__(self, other):
            return int.__add__(self, other) + 1

        def __repr__(self):
            return '<weirdo %d>' % self

        # public API
        def do_this(self):
            print('this')

        def do_that(self):
            print('that')

No user-defined method should use this convention unless it explicitly has to implement
one of the Python object protocols. So don't invent your own dunder methods such as this:

.. code-block:: python

    class BadHabits:
        def __my_method__(self):
            print('ok')

2.3. Arguments
++++++++++++++

Arguments are in lowercase, with underscores if needed. They follow the same naming
rules as variables because arguments are simply local variables that get their value as
function input values. In the following example, ``text`` and ``separator`` are arguments of
``one_line()`` function:

.. code-block:: python

    def one_line(text, separator=" "):
    """Convert possibly multiline text to single line"""
        return separator.join(text.split())

2.4. Properties
+++++++++++++++

The names of properties are in lowercase, or in lowercase with underscores. Most of the
time they represent an object's state, which can be a noun or an adjective, or a small phrase
when needed. In the following code example, the ``Container`` class is a simple data
structure that can return copies of its contents through ``unique_items`` and
``ordered_items`` properties:

.. code-block:: python

    class Container:
        _contents = []
        def append(self, item):
            self._contents.append(item)

        @property
        def unique_items(self):
            return set(self._contents)
        @property
        def ordered_items(self):
            return list(self._contents)

2.5. Classes
++++++++++++

The names of classes are always in ``CamelCase``, and may have a leading underscore when
they are private to a module.

In object-oriented programming classes are used to encapsulate the application state.
Attributes of objects are record of that state. Methods are used to modify that state, convert
it into meaningful values or to produce side effects. This is why class names are often noun
phrases and form a usage logic with the method names that are verb phrases. The following
code example contains a ``Document`` class definition with a single ``save()`` method:

.. code-block:: python

    class Document():
        file_name: str
        contents: str
        ...

        def save(self):
            with open(self.file_name, 'w') as file:
                file.write(self.contents)

Class instances often use the same noun phrases as the document but spelled with
lowercase. So, actual ``Document`` class usage could be as follows:

.. code-block:: python

    new_document = Document()
    new_document.save()

2.6. Modules and packages
+++++++++++++++++++++++++

Besides the special module ``__init__``, the module names are in lowercase. The following
are some examples from the standard library:

- ``os``
- ``sys``
- ``shutil``

The Python standard library does not use underscores for module names to separate words
but they are used commonly in many other projects. When the module is private to the
package, a leading underscore is added. Compiled C or C++ modules are usually named
with an underscore and imported in pure Python modules. Package names follow the same
rules, since they act more like structured modules.