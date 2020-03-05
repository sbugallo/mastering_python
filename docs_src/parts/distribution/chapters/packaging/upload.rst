3. Uploading a package
**********************

Packages would be useless without an organized way to store, upload, and download
them. Python Package Index is the main source of open source packages in the Python
community. Anyone can freely upload new packages and the only requirement is to
register on the PyPI site: `https://pypi.python.org/pypi <https://pypi.python.org/pypi>`_.

You are not, of course, limited to only this index and all Python packaging tools support the
usage of alternative package repositories. This is especially useful for distributing closed
source code among internal organizations or for deployment purposes. Details of such
packaging usage with instructions on how to create your own package index will be
explained in the next chapter. Here we focus mainly on open source uploads to PyPI, with
only little mention on how to specify alternative repositories.

3.1. PyPI: Python Package Index
+++++++++++++++++++++++++++++++

Python Package Index is, as already mentioned, the official source of open source package
distributions. Downloading from it does not require any account or permission. The only
thing you need is a package manager that can download new distributions from PyPI. Your
preferred choice should be ``pip``.

3.1.1. Uploading to PyPI
------------------------

Anyone can register and upload packages to PyPI provided that he or she has an account
registered. Packages are bound to the user, so, by default, only the user that registered the
name of the package is its admin and can upload new distributions. This could be a
problem for bigger projects, so there is an option to mark other users as package
maintainers so that they are able to upload new distributions too.

The easiest way to upload a package is to use the following upload command of
the setup.py script:

.. code-block:: bash

    $ python setup.py <dist-commands> upload

Here, ``<dist-commands>`` is a list of commands that creates distributions to upload. Only
distributions created during the same ``setup.py`` execution will be uploaded to the
repository. So, if you upload source distribution, built distribution, and ``wheel`` package at
once, then you need to issue the following command:

.. code-block:: bash

    $ python setup.py sdist bdist bdist_wheel upload

When uploading using ``setup.py``, you cannot reuse distributions that were already built in
previous command calls and are forced to rebuild them on every upload. This may be
inconvenient for large or complex projects where creation of the actual distribution may
take a considerable amount of time. Another problem of ``setup.py`` upload is that it can
use plain text HTTP or unverified HTTPS connections on some Python versions. This is
why Twine is recommended as a secure replacement for the ``setup.py upload`` command.

Twine is the utility for interacting with PyPI that currently serves only one
purpose: securely uploading packages to the repository. It supports any packaging format
and always ensures that the connection is secure. It also allows you to upload files that
were already created, so you are able to test distributions before release. The following
example usage of twine still requires invoking the ``setup.py`` script for building
distributions:

.. code-block:: bash

    $ python setup.py sdist bdist_wheel
    $ twine upload dist/*

3.1.2. .pypirc
--------------

``.pypirc`` is a configuration file that stores information about Python packages repositories.
It should be located in your home directory. The format for this file is as follows:

.. code-block:: cfg

    [distutils]
    index-servers =
        pypi
        other

    [pypi]
    repository: <repository-url>
    username: <username>
    password: <password>

    [other]
    repository: https://example.com/pypi
    username: <username>
    password: <password>

The ``distutils`` section should have the ``index-servers`` variable that lists all sections
describing all the available repositories and credentials for them. There are only the
following three variables that can be modified for each repository section:

- ``repository``: This is the URL of the package repository (it defaults to `https://pypi.org/ <https://pypi.org>`_).
- ``username``: This is the username for authentication in the given repository.
- ``password``: This is the user password for authentication in the given repository (in plain text).

Note that storing your repository password in plain text may not be the wisest security
choice. You can always leave it blank and you should be prompted for it whenever it is
necessary.

The ``.pypirc`` file should be respected by every packaging tool built for Python. While this
may not be true for every packaging-related utility out there, it is supported by the most
important ones, such as ``pip``, ``twine``, ``distutils`` and ``setuptools``.

3.2. Source packages versus built packages
++++++++++++++++++++++++++++++++++++++++++

There are generally the following two types of distributions for Python packages:

- Source distributions
- Built (binary) distributions

Source distributions are the simplest and most platform independent. For pure Python
packages, it is a no-brainer. Such a distribution contains only Python sources and these
should already be highly portable.

A more complex situation is when your package introduces some extensions written, for
example, in C. Source distributions will still work provided that the package user has
proper development toolchain in his/her environment. This consists mostly of the compiler
and proper C header files. For such cases, the build distribution format may be better suited
because it can provide already built extensions for specific platforms.

3.2.1. sdist
++++++++++++

The ``sdist`` command is the simplest command available. It creates a release tree where
everything that is needed to run the package is copied to. This tree is then archived in one
or many archived files (often, it just creates one tarball). The archive is basically a copy of
the source tree.

This command is the easiest way to distribute a package that would be independent from
the target system. It creates a ``dist/`` directory for storing the archives to be distributed.
Before you create the first distribution, you have to provide a ``setup()`` call with a version
number, as follows. If you don't, ``setuptools`` module will assume default value
of ``version = '0.0.0'``:

.. code-block:: python

    from setuptools import setup
    setup(name='acme.sql', version='0.1.1')

Every time a package is released, the version number should be increased so that the target
system knows the package has changed.

Let's run the following ``sdist`` command for ``acme.sql`` package in ``0.1.1`` version:

.. code-block:: bash

    $ python setup.py sdist
    running sdist
    ...
    creating dist
    tar -cf dist/acme.sql-0.1.1.tar acme.sql-0.1.1
    gzip -f9 dist/acme.sql-0.1.1.tar
    removing 'acme.sql-0.1.1' (and everything under it)

    $ ls dist/
    acme.sql-0.1.1.tar.gz

.. note:: On Windows, the default archive type will be ZIP.

The version is used to mark the name of the archive, which can be distributed and installed
on any system that has Python. In the ``sdist`` distribution, if the package contains C libraries
or extensions, the target system is responsible for compiling them. This is very common for
Linux-based systems or macOS because they commonly provide a compiler. But it is less
usual to have it under Windows. That's why a package should always be distributed with a
prebuilt distribution as well, when it is intended to be run on several platforms.


3.2.2. bdist and wheels
-----------------------

To be able to distribute a prebuilt distribution, ``distutils`` provides the ``build`` command.
This commands compiles the package in the following four steps:

- ``build_py``: This builds pure Python modules by byte-compiling them and copying them into the build folder.
- ``build_clib``: This builds C libraries, when the package contains any, using Python compiler and creating a static library in the build folder.
- ``build_ext``: This builds C extensions and puts the result in the build folder like ``build_clib``.
- ``build_scripts``: This builds the modules that are marked as scripts. It also changes the interpreter path when the first line was set (using ``!#`` prefix) and fixes the file mode so that it is executable.

Each of these steps is a command that can be called independently. The result of the
compilation process is a ``build`` folder that contains everything needed for the package to be
installed. There's no cross-compiler option yet in the ``distutils`` package. This means that
the result of the command is always specific to the system it was built on.

When some C extensions have to be created, the build process uses the default system
compiler and the Python header file (``Python.h``). This include file is available from the time
Python was built from the sources. For a packaged distribution, an extra package for your
system distribution is probably required. At least in popular Linux distributions, it is often
named ``python-dev``. It contains all the necessary header files for building Python
extensions.

The C compiler used in the build process is the compiler that is default for your operating
system. For a Linux-based system or macOS, this would be gcc or clang respectively. For
Windows, Microsoft Visual C++ can be used (there's a free command-line version
available). The open source project MinGW can be used as well. This can be configured
in ``distutils``.

The ``build`` command is used by the ``bdist`` command to build a binary distribution. It
invokes ``build`` and all the dependent commands, and then creates an archive in the same
way as ``sdist`` does.

Let's create a binary distribution for ``acme.sql`` on macOS as follows:

.. code-block:: bash

    $ python setup.py bdist
    running bdist
    running bdist_dumb
    running build
    ...
    running install_scripts
    tar -cf dist/acme.sql-0.1.1.macosx-10.3-fat.tar .
    gzip -f9 acme.sql-0.1.1.macosx-10.3-fat.tar
    removing 'build/bdist.macosx-10.3-fat/dumb' (and everything under it)

    $ ls dist/
    acme.sql-0.1.1.macosx-10.3-fat.tar.gz
    acme.sql-0.1.1.tar.gz

Notice that the newly created archive's name contains the name of the system and the
distribution it was built on (macOS 10.3).

The same command invoked on Windows will create a another system, specific distribution
archive as follows:

.. code-block:: bash

    C:\acme.sql> python.exe setup.py bdist
    ...

    C:\acme.sql> dir dist
    25/02/2008      08:18       <DIR>       .
    25/02/2008      08:18       <DIR>       ..
    25/02/2008      08:24                   16 055 acme.sql-0.1.1.win32.zip
                        1 File(s)   16 055 bytes
                        2 Dir(s)    22 239 752 192 bytes free

If a package contains C code, apart from a source distribution, it's important to release as
many different binary distributions as possible. At the very least, a Windows binary
distribution is important for those who most probably don't have a C compiler installed.

A binary release contains a tree that can be copied directly into the Python tree. It mainly
contains a folder that is copied into Python's ``site-packages`` folder. It may also contain
cached bytecode files (``*.pyc`` files on Python 2 and ``__pycache__/*.pyc`` on Python 3).

The other kind of build distributions are wheels provided by the ``wheel`` package. When
installed (for example, using ``pip``), the wheel package adds a new ``bdist_wheel`` command
to the ``distutils``. It allows creating platform specific distributions (currently only for
Windows, macOS, and Linux) that are better alternatives to normal bdist distributions. It
was designed to replace another distribution format introduced earlier
by ``setuptools`` called eggs. Eggs are now obsolete, so won't be featured in here. The
list of advantages of using wheels is quite long. Here are the ones that are mentioned on the
Python Wheels page (`http://pythonwheels.com/ <http://pythonwheels.com>`_):

- Faster installation for pure Python and native C extension packages
- Avoids arbitrary code execution for installation. (avoids ``setup.py``)
- Installation of a C extension does not require a compiler on Windows, macOS, or Linux.
- Allows better caching for testing and continuous integration.
- Creates ``.pyc`` files as part of the installation to ensure they match the Python interpreter used
- More consistent installs across platforms and machines

According to PyPA's recommendation, wheels should be your default distribution format.
For a very long time, the binary wheels for Linux were not supported, but that has changed
fortunately. Binary wheels for Linux are called manylinux wheels. The process of building
them is unfortunately not as straightforward as for Windows and macOS binary wheels.
For these kind of wheels, PyPA maintains special Docker images that serve as a ready-to-
use build environments. For sources of these images and more information, you can visit
their official repository on GitHub: `https://github.com/pypa/manylinux <https://github.com/pypa/manylinux>`_.