2. Documentation as code
************************

The best way to keep the documentation of your project up to date is to treat it as code and
store it in the same repository as the source code it documents. Keeping documentation
sources with the source code has the following benefits:

- With a proper version control system, you can track all changes that were made
  to the documentation. If you ever wonder if a particular surprising code behavior
  is really a bug or just an old and forgotten feature, you can dive into the history
  of the documentation to trace how the documentation for the specific feature
  evolved over time.
- It is easier to develop different versions of the documentation if the project has to
  be maintained on several parallel branches (for example, for different clients). If
  the source code of the project diverges from the main development branch, so
  does the documentation for it.
- There are many tools that allow you to generate the reference documentation of
  software APIs straight from the comments included in the source code. This is
  one of the best ways to generate documentation for projects that provide APIs for
  other components (for example, in the form of reusable libraries and remote
  services).

The Python language has some unique qualities that make documenting software
extremely easy and fun. The Python community also provides a huge selection of tools that
allow you to create beautiful and usable API reference documentation straight from Python
sources. The foundation for these tools are so-called docstrings.

2.1. Using Python docstrings
++++++++++++++++++++++++++++

Docstrings are special Python string literals that are intended for documenting Python
functions, methods, classes, and modules. If the first statement of the function, method,
class, or module is a string literal, it will automatically become a docstring and be included
as a value of the ``__doc__`` attribute of that related function, method, class, or module.

Many of the code examples here already feature docstrings, but for the sake of
consistency, let's look at a general example of a module that contains all possible types of
docstrings, as follows:

.. code-block:: python

    """Example module with doctrings.

    This is a module that shows all four types of docstrings:
    - module docstring
    - function docstring
    - method docstring
    - class docstring
    """

    def show_module_documentation():
        """Prints module documentation.
        Module documentation is available as global __doc__ attribute.
        This attribute can be accessed and modified at any time.
        """
        print(__doc__)

    class DocumentedClass:
        """Class that showcases method documentation.
        """

        def __init__(self):
            """Initialize class instance.
            Interesting note: docstrings are valid statements.
            It means that if function or method doesn't have to
            do nothing and has docstring it doesn't have to
            feature any other statements.
            Such no-op functions are useful for defining abstract
            methods or providing implementation stubs that have
            to be implemented later.
            """

Python also provides a ``help()`` function, which is an entry point for the built-in help
system. It is intended for interactive use within the interactive interpreter session in a
similar way as viewing system manual pages using the UNIX ``man`` command. If you provide
a module instance as an input argument to the ``help()`` function, it will format all
docstrings of that module's objects in a tree-like structure. The following is an example of
``help()`` output for the module we presented in the previous code snippet:

.. code-block:: none

    Help on module docexample:

    NAME
        docexample - Example module with doctrings.

    FILE
        /Users/sbugallo/docexample.py

    DESCRIPTION
        This is a module that shows all four types of docstrings:
            - module docstring
            - function docstring
            - method docstring
            - class docstring

    CLASSES
        DocumentedClass

        class DocumentedClass
        |   Class that showcases method documentation.
        |
        |   Methods defined here:
        |
        |   __init__(self)
        |       Initialize class instance.
        |
        |       Interesting note: docstrings are valid statements.
        |       It means that if function or method doesn't have to
        |       do nothing and has docstring it doesn't have to
        |       feature any other statements.
        |
        |       Such no-op functions are useful for defining abstract
        |       methods or providing implementation stubs that have
        |       to be implemented later.

    FUNCTIONS
        show_module_documentation()
            Prints module documentation.

            Module documentation is available as global __doc__ attribute.
            This attribute can be accessed and modified at any time.

2.2. Popular markup languages and styles for documentation
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Inside docstring, you can put whatever you like in any form you like. There is, of course,
the official PEP 257 (Docstring Conventions) document, which is a general guideline for
docstring conventions, but it concentrates mainly on normalized formatting of multiline
string literals for documentation purposes and does not enforce any markup language.

Anyway, if you want to have nice and usable documentation, it is a good thing to decide
on some formalized markup language to use in your docstrings, especially if you plan to
use some kind of documentation generation tool. Proper markup allows documentation
generators to provide code highlighting, do advanced text formatting, include hyperlinks to
other documents and functions, or even include non-textual assets like images of
automatically generated class diagrams.

The best markup language is easy to write and is also readable in raw textual form outside
of the autogenerated reference documentation. It is best if it can be easily used to provide
longer documentation sources for documents living outside of Python docstrings. One of
the most common markup languages designed specifically for Python with these goals in
mind is reStructuredText. It is used by the Sphinx documentation system and is a markup
language used to create official Python language documentation.

Other popular choices for lightweight text markup languages for docstrings are Markdown
and AsciiDoc. The former is particularly popular within the community of GitHub users
and is the most common documentation markup language in general. It is also often
supported out of the box by various tools for self-documenting web APIs.
