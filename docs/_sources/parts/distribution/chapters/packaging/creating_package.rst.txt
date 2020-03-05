1. Creating a package
*********************

Python packaging can be a bit overwhelming at first. The main reason for that is the
confusion about proper tools for creating Python packages. Anyway, once you create your
first package, you will see that this is not as hard as it looks. Also, knowing proper, state-of-
the art packaging tools helps a lot.

You should know how to create packages even if you are not interested in distributing your
code as open source. Knowing how to make your own packages will give you more insight
in the packaging ecosystem and will help you to work with third-party code that is
available on PyPI that you are probably already using.

Also, having your closed source project or its components available as source distribution
packages can help you to deploy your code in different environments. The advantages of
leveraging the Python packaging ecosystem in the code deployment process will be
described in more detail in the next chapter. Here we will focus on proper tools and
techniques to create such distributions.

1.1. The confusing state of Python packaging tools
++++++++++++++++++++++++++++++++++++++++++++++++++

The state of Python packaging was very confusing for a long time and it took many years to
bring organization to this topic. Everything started with the ``distutils`` package introduced in
1998, which was later enhanced by setuptools in 2003. These two projects started a long and
knotted story of forks, alternative projects, and complete rewrites that tried to (once and for
all) fix the Python packaging ecosystem. Unfortunately, most of these attempts never
succeeded. The effect was quite the opposite. Each new project that aimed to
supersede ``setuptools`` or ``distutils`` only added to the already huge confusion around
packaging tools. Some of such forks were merged back to their ancestors (such as
``distribute`` which was a fork of ``setuptools``) but some were left abandoned (such as
``distutils2``).

Fortunately, this state is gradually changing. An organization called the Python Packaging
Authority (PyPA) was formed to bring back the order and organization to the packaging
ecosystem. The Python Packaging User Guide (`https://packaging.python.org <https://packaging.python.org>`_),
maintained by PyPA, is the authoritative source of information about the latest packaging
tools and best practices. Treat that site as the best source of information about packaging
and complementary reading for this chapter. This guide also contains a detailed history of
changes and new projects related to packaging. So it is worth reading it, even if you already
know a bit about packaging, to make sure you still use the proper tools.

Stay away from other popular internet resources, such as "The Hitchhiker's Guide to
Packaging". It is old, not maintained, and mostly obsolete. It may be interesting only for
historical reasons, and the Python Packaging User Guide is in fact a fork of this old
resource.


1.1.1. The current landscape of Python packaging thanks to PyPA
---------------------------------------------------------------

PyPA, besides providing an authoritative guide for packaging, also maintains packaging
projects and a standardization process for new official aspects of Python packaging. All of
PyPA's projects can be found under a single organization on GitHub:
`https://github.com/pypa <https://github.com/pypa>`_

Some of them were already mentioned. The following are the most notable:

- ``pip``
- ``virtualenv``
- ``twine``
- ``warehouse``

Note that most of them were started outside of this organization and were moved under
PyPA patronage when they become mature and widespread solutions.

Thanks to PyPA engagement, the progressive abandonment of the eggs format in favor of
wheels for built distributions has already happened. Also thanks to the commitment of the
PyPA community, the old PyPI implementation was finally totally rewritten in the form of
the Warehouse project. Now, PyPI has got a modernized user interface and many long-
awaited usability improvements and features.

1.1.2. Tool recommendations
---------------------------

The Python Packaging User Guide gives a few suggestions on recommended tools for
working with packages. They can be generally divided into the following two groups:

- Tools for installing packages
- Tools for package creation and distribution

Utilities from the first group recommended are:

- Use ``pip`` for installing packages from PyPI.
- Use ``virtualenv`` or ``venv`` for application-level isolation of the Python runtime environment.

The Python Packaging User Guide recommendations of tools for package creation and
distribution are as follows:

- Use ``setuptools`` to define projects and create **source distributions**.
- Use **wheels** in favor of **eggs** to create **built distributions**.
- Use ``twine`` to upload package distributions to PyPI.

1.2. Project configuration
++++++++++++++++++++++++++

It should be obvious that the easiest way to organize the code of big applications is to split
them into several packages. This makes the code simpler, easier to understand, maintain,
and change. It also maximizes the reusability of your code. Separate packages act as
components that can be used in various programs.

1.2.1. setup.py
---------------

The root directory of a package that has to be distributed contains a ``setup.py`` script. It
defines all metadata as described in the ``distutils`` module. Package metadata is expressed
as arguments in a call to the standard ``setup()`` function. Despite ``distutils`` being the
standard library module provided for the purpose of code packaging, it is actually
recommended to use the ``setuptools`` instead. The ``setuptools`` package provides several
enhancements over the standard ``distutils`` module.

Therefore, the minimum content for this file is as follows:

.. code-block:: python

    from setuptools import setup

    setup(
        name='mypackage'
    )

``name`` gives the full name of the package. From there, the script provides several commands
that can be listed with the ``--help-commands`` option, as shown in the following code:

.. code-block:: bash

    $ python3 setup.py --help-commands
    Standard commands:
        build           build everything needed to install
        clean           clean up temporary files from 'build' command
        install         install everything from build directory
        sdist           create a source distribution (tarball, zip file, etc.)
        register        register the distribution with the Python package index
        bdist           create a built (binary) distribution
        check           perform some checks on the package
        upload          upload binary package to PyPI

    Extra commands:
        bdist_wheel     create a wheel distribution
        alias           define a shortcut to invoke one or more commands
        develop         install package in 'development mode'

    usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
    or: setup.py --help [cmd1 cmd2 ...]
    or: setup.py --help-commands
    or: setup.py cmd --help

The actual list of commands is longer and can vary depending on the available
``setuptools`` extensions. It was truncated to show only those that are most important and
relevant to this chapter. **Standard commands** are the built-in commands provided
by ``distutils``, whereas **extra commands** are the ones provided by third-party packages,
such as ``setuptools`` or any other package that defines and registers a new command.
Here, one such extra command registered by another package is ``bdist_wheel``, provided
by the ``wheel`` package.

1.2.2. setup.cfg
----------------

The ``setup.cfg`` file contains default options for commands of the ``setup.py`` script. This is
very useful if the process for building and distributing the package is more complex and
requires many optional arguments to be passed to the ``setup.py`` script commands.
This ``setup.cfg`` file allows you to store such default parameters together with your source
code on a per project basis. This will make your distribution flow independent from the
project and also provides transparency about how your package was built/distributed to
the users and other team members.

The syntax for the ``setup.cfg`` file is the same as provided by the built-in
``configparser`` module so it is similar to the popular Microsoft Windows INI files. Here
is an example of the ``setup.cfg`` configuration file that provides some ``global``, ``sdist``,
and ``bdist_wheel`` commands' defaults:

.. code-block:: cfg

    [global]
    quiet=1

    [sdist]
    formats=zip,tar

    [bdist_wheel]
    universal=1

This example configuration will ensure that source distributions (``sdist`` section) will
always be created in two formats (ZIP and TAR) and the built ``wheel`` distributions
(``bdist_wheel`` section) will be created as universal wheels that are independent from the
Python version. Also most of the output will be suppressed on every command by the
global ``--quiet`` switch. Note that this option is included here only for demonstration
purposes and it may not be a reasonable choice to suppress the output for every command
by default.

1.2.3. MANIFEST.in
------------------

When building a distribution with the ``sdist`` command, the ``distutils`` module browses
the package directory looking for files to include in the archive. By default ``distutils`` will
include the following:

- All Python source files implied by the ``py_modules``, ``packages``, and ``scripts`` arguments
- All C source files listed in the ``ext_modules`` argument
- Files that match the glob pattern ``test/test*.py``
- Files named ``README``, ``README.txt``, ``setup.py``, and ``setup.cfg``

Besides that, if your package is versioned with a version control system such as Subversion,
Mercurial, or Git, there is the possibility to auto-include all version controlled files using
additional ``setuptools`` extensions such as ``setuptools-svn``, ``setuptools-hg``,
and ``setuptools-git``. Integration with other version control systems is also possible
through other custom extensions. No matter if it is the default built-in collection strategy or
one defined by custom extension, the ``sdist`` will create a ``MANIFEST`` file that lists all files
and will include them in the final archive.

Let's say you are not using any extra extensions, and you need to include in your package
distribution some files that are not captured by default. You can define a template
called ``MANIFEST.in`` in your package root directory (the same directory as ``setup.py`` file).
This template directs the ``sdist`` command on which files to include.

This ``MANIFEST.in`` template defines one inclusion or exclusion rule per line:

.. code-block:: cfg

    include HISTORY.txt
    include README.txt
    include CHANGES.txt
    include CONTRIBUTORS.txt
    include LICENSE
    recursive-include *.txt *.py

The full list of the ``MANIFEST.in`` commands can be found in the official ``distutils``
documentation.

1.2.4. Most important metadata
------------------------------

Besides the name and the version of the package being distributed, the most important
arguments that the ``setup()`` function can receive are as follows:

- ``description``: This includes a few sentences to describe the package.
- ``long_description``: This includes a full description that can be in reStructuredText (default) or other supported markup languages.
- ``long_description_content_type``: this defines MIME type of long description; it is used to tell the package repository what kind of markup language is used for the package description.
- ``keywords``: This is a list of keywords that define the package and allow for better indexing in the package repository.
- ``author``: This is the name of the package author or organization that takes care of it.
- ``author_email``: This is the contact email address.
- ``url``: This is the URL of the project.
- ``license``: This is the name of the license (GPL, LGPL, and so on) under which the package is distributed.
- ``packages``: This is a list of all package names in the package distribution; ``setuptools`` provides a small function called ``find_packages`` that can automatically find package names to include.
- ``namespace_packages``: This is a list of namespace packages within package distribution.

1.2.5. Trove classifiers
------------------------

PyPI and ``distutils`` provide a solution for categorizing applications with the set of
classifiers called trove classifiers. All trove classifiers form a tree-like structure. Each
classifier string defines a list of nested namespaces where every namespace is separated by
the :: substring. Their list is provided to the package definition as
a classifiers argument of the ``setup()`` function.

Here is an example list of classifiers taken from ``solrq`` project available on PyPI:


.. code-block:: python

    from setuptools import setup


    setup(
        name="solrq",
        # (...)
        classifiers=[
        'Development Status :: 4 - Beta',
        'Intended Audience :: Developers',
        'License :: OSI Approved :: BSD License',
        'Operating System :: OS Independent',
        'Programming Language :: Python',
        'Programming Language :: Python :: 2',
        'Programming Language :: Python :: 2.6',
        'Programming Language :: Python :: 2.7',
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.2',
        'Programming Language :: Python :: 3.3',
        'Programming Language :: Python :: 3.4',
        'Programming Language :: Python :: Implementation :: PyPy',
        'Topic :: Internet :: WWW/HTTP :: Indexing/Search',
        ]
    )

Trove classifiers are completely optional in the package definition but provide a useful
extension to the basic metadata available in the ``setup()`` interface. Among others, trove
classifiers may provide information about supported Python versions, supported operating
systems, the development stage of the project, or the license under which the code is
released. Many PyPI users search and browse the available packages by categories so a
proper classification helps packages to reach their target.

Trove classifiers serve an important role in the whole packaging ecosystem and should
never be ignored. There is no organization that verifies packages classification, so it is your
responsibility to provide proper classifiers for your packages and not introduce chaos to the
whole package index.

At the time of writing this section, there are 667 classifiers available on PyPI that are grouped
into the following nine major categories:

- Development status
- Environment
- Framework
- Intended audience
- License
- Natural language
- Operating system
- Programming language
- Topic

This list is ever-growing, and new classifiers are added from time to time. It is thus possible
that the total count of them will be different at the time you read this. The full list of
currently available trove classifiers is available at `https://pypi.org/classifiers <https://pypi.org/classifiers>`_.

1.2.6. Common patterns
----------------------

Creating a package for distribution can be a tedious task for unexperienced developers.
Most of the metadata that ``setuptools`` or ``distutils`` accept in their ``setup()`` function call
can be provided manually ignoring the fact that this metadata may be also available in
other parts of the project. Here is an example:


.. code-block:: python

    from setuptools import setup


    setup(
        name="myproject",
        version="0.0.1",
        description="mypackage project short description",
        long_description="""
            Longer description of mypackage project
            possibly with some documentation and/or
            usage examples
        """,
        install_requires=[
            'dependency1',
            'dependency2',
            'etc'
        ]
    )

Some of the metadata elements are often found in different places in a typical Python
project. For instance, content of long description is commonly included in the project's
``README`` file, and it is a good convention to put a version specifier in the ``__init__`` module
of the package. Hardcoding such package metadata as ``setup()`` function
arguments redundancy to the project that allows for easy mistakes and inconsistencies in
future. Both ``setuptools`` and ``distutils`` cannot automatically pick metadata information
from the project sources, so you need to provide it yourself. There are some common
patterns among the Python community for solving the most popular problems such as
dependency management, version/readme inclusion, and so on. It is worth knowing at least
a few of them because they are so popular that they could be considered as packaging
idioms.

1.2.6.1. Automated inclusion of version string from package
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The PEP 440 "Version Identification and Dependency Specification" document specifies a standard
for version and dependency specification. It is a long document that covers accepted
version specification schemes and defines how version matching and comparison in Python
packaging tools should work. If you are using or plan to use a complex project version
numbering scheme, then you should definitely read this document carefully. If you are
using a simple scheme that consists just of one, two, three, or more numbers separated by
dots, then you don't have to dig into the details of PEP 440. If you don't know how to
choose the proper versioning scheme, I greatly recommend following the semantic
versioning scheme.

The other problem related to code versioning is where to include that version specifier for a
package or module. There is PEP 396 (Module Version Numbers) that deals exactly with
this problem. PEP 396 is only an informational document and has a deferred status, so it is
not a part of the official Python standards track. Anyway, it describes what seems to be a
*de facto* standard now. According to PEP 396, if a package or module has a specific version
defined, the version specifier should be included as a ``__version__`` attribute of package
root ``__init__.py`` INI file or distributed module file. Another *de facto* standard is to also
include the ``VERSION`` attribute that contains the tuple of the version specifier parts. This
helps users to write compatibility code because such version tuples can be easily compared
if the versioning scheme is simple enough.

So many packages available on PyPI follow both conventions. Their ``__init__.py`` files
contain version attributes that look like the following:

.. code-block:: python

    VERSION = (0, 1, 1)
    __version__ = ".".join([str(x) for x in VERSION])

The other suggestion of PEP 396 is that the version argument provided in
the ``setup()`` function of the ``setup.py`` script should be derived from ``__version__``, or the
other way around. The Python Packaging User Guide features multiple patterns for single-
sourcing project versioning, and each of them has its own advantages and limitations. My
personal favorite is rather long and is not included in the PyPA's guide, but has the
advantage of limiting the complexity only to the ``setup.py`` script. This boilerplate assumes
that the version specifier is provided by the ``VERSION`` attribute of the
package's ``__init__`` module and extracts this data for inclusion in the ``setup()`` call. Here
is an excerpt from some imaginary package's setup.py script that illustrates this approach:

.. code-block:: python

    from setuptools import setup
    import os

    def get_version(version_tuple):
        if not isinstance(version_tuple[-1], int):
            return '.'.join(map(str, version_tuple[:-1])) + version_tuple[-1]
        return '.'.join(map(str, version_tuple))

    init = os.path.join(os.path.dirname(__file__), 'src', 'some_package', '__init__.py')
    version_line = list(filter(lambda l: l.startswith('VERSION'), open(init)))[0]
    PKG_VERSION = get_version(eval(version_line.split('=')[-1]))

    setup(
        name='some-package',
        version=PKG_VERSION,
        # ...
    )

1.2.6.2. README file
~~~~~~~~~~~~~~~~~~~~

The Python Package Index can display the project's README file or the value of
``long_description`` on the package page in the PyPI portal. PyPI is able to interpret the
markup used in the ``long_description`` content and render it as HTML on the package
page. The type of markup language is controlled through
the ``long_description_content_type`` argument of the ``setup()`` call. For now, there are
the following three choices for markup available:

- Plain text with ``long_description_content_type='text/plain'``
- reStructuredText with ``long_description_content_type='text/x-rst'``
- Markdown with ``long_description_content_type='text/markdown'``

Markdown and reStructuredText are the most popular choices among Python developers,
but some might still want to use different markup languages for various reasons. If you
want to use something different as your markup language for your project's README, you
can still provide it as a project description on the PyPI page in a readable form. The trick
lies in using the ``pypandoc`` package to translate your other markup language into
reStructuredText (or Markdown) while uploading the package to the Python Package
Index. It is important to do it with a fallback to plain content of your README file, so the
installation won't fail if the user has no ``pypandoc`` installed. The following is an example of
a ``setup.py`` script that is able to read the content of the README file written in AsciiDoc
markup language and translate it to reStructuredText before including
a ``long_description`` argument:

.. code-block:: python

    from setuptools import setup


    try:
        from pypandoc import convert

        def read_md(file_path):
            return convert(file_path, to='rst', format='asciidoc')

    except ImportError:
        convert = None
        print("warning: pypandoc module not found, could not convert Asciidoc to RST")

        def read_md(file_path):
            with open(file_path, 'r') as f:
            return f.read()

    README = os.path.join(os.path.dirname(__file__), 'README')

    setup(
        name='some-package',
        long_description=read_md(README),
        long_description_content_type='text/x-rst',
        # ...
    )

1.2.6.3. Managing dependencies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Many projects require some external packages to be installed in order to work properly.
When the list of dependencies is very long, there comes a question as to how to manage it.
The answer in most cases is very simple. Do not over-engineer it. Keep it simple and
provide the list of dependencies explicitly in your setup.py script as follows:

.. code-block:: python

    from setuptools import setup


    setup(
        name='some-package',
        install_requires=['falcon', 'requests', 'delorean']
        # ...
    )

Some Python developers like to use ``requirements.txt`` files for tracking lists of
dependencies for their packages. In some situations, you might find some reason for doing
that, but in most cases, this is a relic of times where the code of that project was not
properly packaged. Anyway, even such notable projects as Celery still stick to this
convention. So if you are not willing to change your habits or you are somehow forced to
use requirement files, then at least do it properly. Here is one of the popular idioms for
reading the list of dependencies from the ``requirements.txt`` file:

.. code-block:: python

    from setuptools import setup
    import os

    def strip_comments(l):
        return l.split('#', 1)[0].strip()

    def reqs(*f):
        return list(filter(None, [strip_comments(l)
                                  for l in open(os.path.join(os.getcwd(), *f)).readlines()]))
    setup(
        name='some-package',
        install_requires=reqs('requirements.txt')
        # ...
    )

1.2.7. The custom setup command
+++++++++++++++++++++++++++++++

``distutils`` allows you to create new commands. A new command can be registered with
an entry point, which was introduced by ``setuptools`` as a simple way to define packages
as plugins.

An entry point is a named link to a class or a function that is made available through some
APIs in ``setuptools``. Any application can scan for all registered packages and use the
linked code as a plugin.

To link the new command, the entry_points metadata can be used in the setup call as
follows:

.. code-block:: python

    setup(
        name="my.command",
        entry_points="""
            [distutils.commands]
            my_command = my.command.module.Class
        """
    )

All named links are gathered in named sections. When ``distutils`` is loaded, it scans for
links that were registered under ``distutils.commands``.

This mechanism is used by numerous Python applications that provide extensibility.

1.3. Working with packages during development
+++++++++++++++++++++++++++++++++++++++++++++

Working with ``setuptools`` is mostly about building and distributing packages. However,
you still need to use ``setuptools`` to install packages directly from project sources. And the
reason for that is simple. It is a good habit to test if our packaging code works properly
before submitting your package to PyPI. And the simplest way to test it is by installing it. If
you send a broken package to the repository, then in order to re-upload it, you need to
increase the version number.

Testing if your code is packaged properly before the final distribution saves you from
unnecessary version number inflation and obviously from wasting your time. Also,
installation directly from your own sources using ``setuptools`` may be essential when
working on multiple related packages at the same time.

1.3.1. setup.py install
-----------------------

The ``install`` command installs the package in your current Python environment. It will try
to build the package if no previous build was made and then inject the result into the
filesystem directory where Python is looking for installed packages. If you have an archive
with a source distribution of some package, you can decompress it in a temporary folder
and then install it with this command. The ``install`` command will also install
dependencies that are defined in the ``install_requires`` argument. Dependencies will be
installed from the Python Package Index.

An alternative to the bare ``setup.py`` script when installing a package is to use ``pip``. Since it
is a tool that is recommended by PyPA, you should use it even when installing a package in
your local environment just for development purposes. In order to install a package from
local sources, run the following command:

.. code-block:: bash

    pip install <project-path>

1.3.2. Uninstalling packages
----------------------------

Amazingly, ``setuptools`` and ``distutils`` lack the ``uninstall`` command. Fortunately, it is
possible to uninstall any Python package using ``pip`` as follows:

.. code-block:: bash

    pip uninstall <package-name>

Uninstalling can be a dangerous operation when attempted on system-wide packages. This
is another reason why it is so important to use virtual environments for any development.

1.3.3. setup.py develop or pip -e
---------------------------------

Packages installed with ``setup.py`` install are copied to the site-packages directory of
your current Python environment. This means that whenever you make a change to the
sources of that package, you are required to reinstall it. This is often a problem during
intensive development because it is very easy to forget about the need to perform
installation again. This is why ``setuptools`` provides an extra ``develop`` command that
allows you to install packages in the development mode. This command creates a special
link to project sources in the deployment directory (site-packages) instead of copying
the whole package there. Package sources can be edited without the need for reinstallation
and are available in the ``sys.path`` as if they were installed normally.

``pip`` also allows you to install packages in such a mode. This installation option is
called editable mode and can be enabled with the ``-e`` parameter in the ``install`` command
as follows:

.. code-block:: bash

    pip install -e <project-path>

Once you install the package in your environment in editable mode, you can freely modify
the installed package in place and all the changes will be immediately visible without the
need to reinstall the package.
