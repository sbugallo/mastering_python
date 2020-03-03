7. Useful tools
***************

Common conventions and practices used in a software project should always be
documented. But having proper documentation for guidelines is often not enough to
enforce that these guidelines are actually followed. Fortunately, you can use automated
tools that can check sources of your code and verify if it meets specific naming conventions
and style guidelines.

The following are a few popular tools:

- ``pylint``: This is a very flexible source code analyzer
- ``pycodestyle`` and ``flake8``: This is a small code style checker and a wrapper that adds to it some more useful features, such as static analysis and complexity measurement

7.1. Pylint
+++++++++++

Besides some quality assurance metrics, Pylint allows for checking of whether a given
source code is following a naming convention. Its default settings correspond to PEP 8 and
a Pylint script provides a shell report output.

To install Pylint, you can use ``pip`` as follows:

.. code-block:: bash

    $ pip install pylint

After this step, the command is available and can be run against a module, or several
modules using wildcards. Let's try it on Buildout's ``bootstrap.py`` script as follows:

.. code-block:: bash

    $ wget -O bootstrap.py https://bootstrap.pypa.io/bootstrap-buildout.py -q
    $ pylint bootstrap.py
    No config file found, using default configuration
    ************* Module bootstrap
    C: 76, 0: Unnecessary parens after 'print' keyword (superfluous-parens)
    C: 31, 0: Invalid constant name "tmpeggs" (invalid-name)
    C: 33, 0: Invalid constant name "usage" (invalid-name)
    C: 45, 0: Invalid constant name "parser" (invalid-name)
    C: 74, 0: Invalid constant name "options" (invalid-name)
    C: 74, 9: Invalid constant name "args" (invalid-name)
    C: 84, 4: Import "from urllib.request import urlopen" should be placed at
    the top of the module (wrong-import-position)
    ...
    Global evaluation
    -----------------
    Your code has been rated at 6.12/10

Real Pylint's output is a bit longer and here it has been truncated for the sake of brevity.

Remember that Pylint can often give you false positive warnings that decrease the overall
quality rating. For instance, an import statement that is not used by the code of the module
itself is perfectly fine in some cases (for example, building top-level ``__init__`` modules in a
package). Always treat Pylint's output as a hint and not an oracle.

Making calls to libraries that are using mixedCase for methods can also lower your rating.
In any case, the global evaluation of your code score is not that important. Pylint is just a
tool that points you to places where there is the possibility for improvements.

It is always recommended to do some tuning of Pylint. In order to do so you need to create
a ``.pylinrc`` configuration file in your project's root directory. You can do that using the
following ``-generate-rcfile`` option of the ``pylint`` command:

.. code-block:: bash

    $ pylint --generate-rcfile > .pylintrc

This configuration file is self-documenting (every possible option is described with
comment) and should already contain every available Pylint configuration option.

Besides checking for compliance with some arbitrary coding standards, Pylint can also give
additional information about the overall code quality, such as:

- Code duplication metrics
- Unused variables and imports
- Missing function, method, or class docstrings
- Too long function signatures

The list of available checks that are enabled by default is very long. It is important to know
that some of the rules are very arbitrary and cannot always be easily applied to every code
base. Remember that consistency is always more valuable than compliance to some
arbitrary rules. Fortunately, Pylint is very tunable, so if your team uses some naming and
coding conventions that are different from the ones assumed by default, you can easily
configure Pylint to check for consistency with your own conventions.

7.2. pycodestyle and flake8
+++++++++++++++++++++++++++

``pycodestyle`` (formerly ``pep8``) is a tool that has only one purpose; it provides only style
checking against code conventions defined in PEP 8. This is the main difference from Pylint
that has many more additional features. This is the best option for programmers that are
interested in automated code style checking only for the PEP 8 standard, without any
additional tool configuration, as in Pylint's case.

``pycodestyle`` can be installed with pip as follows:

.. code-block:: bash

    $ pip install pycodestyle

When run on the Buildout's ``bootstrap.py`` script, it will give the following short list of
code style violations:

.. code-block:: bash

    $ wget -O bootstrap.py https://bootstrap.pypa.io/bootstrap-buildout.py -q
    $ pycodestyle bootstrap.py
    bootstrap.py:118:1: E402 module level import not at top of file
    bootstrap.py:119:1: E402 module level import not at top of file
    bootstrap.py:190:1: E402 module level import not at top of file
    bootstrap.py:200:1: E402 module level import not at top of file

The main difference from Pylint's output is its length. ``pycodestyle`` concentrates only on
style, so it does not provide any other warnings, such as unused variables, too long
function names, or missing docstrings. It also does not give a rating. And it really makes
sense because there is no such thing as partial consistency or partial conformance. Any,
even the slightest, violation of style guidelines makes the code immediately inconsistent.

The code of ``pycodestyle`` is simpler than Pylint's and its output is easier to parse, so it may
be a better choice if you want to make your code style verification part of a continuous
integration process. If you are missing some static analysis features, there is
the ``flake8`` package that is a wrapper on ``pycodestyle`` and a few other tools that are easily
extendable and provide a more extensive suite of features. These include the following:

- McCabe complexity measurement
- Static analysis via ``pyflakes``
- Disabling whole files or single lines using comments
