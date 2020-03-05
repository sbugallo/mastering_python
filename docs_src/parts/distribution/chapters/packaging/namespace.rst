2. Namespace packages
*********************

*The Zen of Python* that you can read after writing ``import this`` in the interpreter session
says the following about namespaces:

*"Namespaces are one honking great idea-let's do more of those!"*

And this can be understood in at least two ways. The first is a namespace in context of the
language. We all use the following namespaces without even knowing:

- The global namespace of a module
- The local namespace of the function or method invocation
- The class namespace

The other kind of namespaces can be provided at the packaging level. These are **namespace
packages**. This is often an overlooked feature of Python packaging that can be very useful
in structuring the package ecosystem in your organization or in a very large project.

Namespace packages can be understood as a way of grouping related packages, where each
of these packages can be installed independently.

Namespace packages are especially useful if you have components of your application
developed, packaged, and versioned independently but you still want to access them from
the same namespace. This also helps to make clear to which organization or project every
package belongs. For instance, for some imaginary Acme company, the common
namespace could be ``acme``. Therefore this organization could create the
general ``acme`` namespace package that could serve as a container for other packages from
this organization. For example, if someone from Acme wants to contribute to this
namespace with, for example, an SQL-related library, they can create a
new ``acme.sql`` package that registers itself in the acme namespace.

It is important to know what's the difference between normal and namespace packages and
what problem they solve. Normally (without namespace packages), you would create a
package called ``acme`` with an ``sql`` subpackage/submodule with the following file structure:

.. code-block:: bash

    $ tree acme/
    acme/
    ├── acme
    │    ├── __init__.py
    │    └── sql
    │       └── __init__.py
    └── setup.py

    2 directories, 3 files

Whenever you want to add a new subpackage, let's say ``templating``, you are forced to
include it in the source tree of ``acme`` as follows:

.. code-block:: bash

    $ tree acme/
    acme/
    ├── acme
    │   ├── __init__.py
    │   ├── sql
    │   │    └── __init__.py
    │   └── templating
    │        └── __init__.py
    └── setup.py

    3 directories, 4 files

Such an approach makes independent development
of ``acme.sql`` and ``acme.templating`` almost impossible. The ``setup.py`` script will also
have to specify all dependencies for every subpackage. So it is impossible (or at least very
hard) to have an installation of some of the ``acme`` components optional. Also, with enough
subpackages it is practically impossible to avoid dependency conflicts.

With namespace packages, you can store the source tree for each of these subpackages
independently as follows:

.. code-block:: bash

    $ tree acme.sql/
    acme.sql/
    ├── acme
    │    └── sql
    │       └── __init__.py
    └── setup.py

    2 directories, 2 files


    $ tree acme.templating/
    acme.templating/
    ├── acme
    │   └── templating
    │       └── __init__.py
    └── setup.py

    2 directories, 2 files

And you can also register them independently in PyPI or any package index you use. Users
can choose which of the subpackages they want to install from the acme namespace as
follows, but they never install the general ``acme`` package (it doesn't even have to exist):

.. code-block:: bash

    $ pip install acme.sql acme.templating

Note that independent source trees are not enough to create namespace packages in
Python. You need a bit of additional work if you don't want your packages to not overwrite
each other. Also proper handling may be different depending on the Python language
version you target. Details of that are described in the next two sections.

2.1. Implicit namespace packages
++++++++++++++++++++++++++++++++

If you use and target only Python 3, then there is good news for you. PEP 420 (Implicit
Namespace Packages) introduced a new way to define namespace packages. It is part of
the standards track and became an official part of the language since version 3.3. In short,
every directory that contains Python packages or modules (including namespace packages
too) is considered a namespace package if it does not contain the ``__init__.py`` file. So, the
following are examples of file structures presented in the previous section:

.. code-block:: bash

    $ tree acme.sql/
    acme.sql/
    ├── acme
    │    └── sql
    │       └── __init__.py
    └── setup.py

    2 directories, 2 files


    $ tree acme.templating/
    acme.templating/
    ├── acme
    │   └── templating
    │       └── __init__.py
    └── setup.py

    2 directories, 2 files

They are enough to define that acme is a namespace package under Python 3.3 and later.
Minimal ``setup.py`` for ``acme.templating`` package will look like following:

.. code-block:: python

    from setuptools import setup
    setup(
        name='acme.templating',
        packages=['acme.templating'],
    )

Unfortunately, the ``setuptools.find_packages()`` function does not support PEP 420 at
the time of writing this section. This may change in the future. Also, a requirement to
explicitly define a list of packages seems to be a very small price to pay for easy integration
of namespace packages.

2.2. Namespace packages in previous Python versions
+++++++++++++++++++++++++++++++++++++++++++++++++++

You can't use implicit namespace packages (PEP 420 layout) in Python versions older than
3.3. Still, the concept of namespace packages is very old and was commonly used for years
in such mature projects such as Zope. It means that it is definitely possible to use
namespace packages in older version of Python. Actually, there are several ways to define
that the package should be treated as a namespace.

The simplest one is to create a file structure for each component that resembles an ordinary
package layout without implicit namespace packages and leave everything to ``setuptools``.

So, the example layout for ``acme.sql`` and a ``cme.templating`` could be the following:

.. code-block:: bash

    $ tree acme.sql/
    acme.sql/
    ├── acme
    │    ├── __init__.py
    │    └── sql
    │       └── __init__.py
    └── setup.py

    2 directories, 3 files


    $ tree acme.templating/
    acme.templating/
    ├── acme
    │    ├── __init__.py
    │    └── templating
    │       └── __init__.py
    └── setup.py

    2 directories, 3 files

Note that for both ``acme.sql`` and ``acme.templating``, there is an additional source
file, ``acme/__init__.py``. This file must be left empty. The ``acme`` namespace package will be
created if we provide its name as a value of the ``namespace_packages`` keyword argument
of the ``setuptools.setup()`` function as follows:

.. code-block:: python

    from setuptools import setup


    setup(
        name='acme.templating',
        packages=['acme.templating'],
        namespace_packages=['acme'],
    )

Easiest does not mean best. The ``setuptools`` module in order to register a new namespace
will call for the ``pkg_resources.declare_namespace()`` function in
your ``__init__.py`` file. It will happen even if the ``__init__.py`` file is empty. Anyway, as
the official documentation says, it is your own responsibility to declare namespaces in
the ``__init__.py`` file, and this implicit behavior of ``setuptools`` may be dropped in the
future. In order to be safe and future-proof, you need to add the following line to
the ``acme/__init__.py`` file:

.. code-block:: python

    __import__('pkg_resources').declare_namespace(__name__)

This line will make your namespace package safe from potential future changes regarding
namespace packages in the ``setuptools`` module.
