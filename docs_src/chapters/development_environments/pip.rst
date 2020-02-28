1. Installing packages with pip
*******************************

Nowadays, a lot of operating systems come with Python as a standard component. Most
Linux distributions and UNIX-based systems, such as FreeBSD, NetBSD, OpenBSD, or
macOS, come with Python either installed by default or available through system package
repositories. Many of them even use it as part of some core components: Python powers
the installers of Ubuntu (Ubiquity), Red Hat Linux (Anaconda), and Fedora (Anaconda
again). Unfortunately, the preinstalled system version of Python is often Python 2.7, which
is fairly outdated.

Due to Python's popularity as an operating system component, a lot of packages from PyPI
are also available as native packages managed by the system's package management tools,
such as apt-get (Debian, Ubuntu), rpm (Red Hat Linux), or emerge (Gentoo). It should be
remembered, however, that the list of available libraries is very limited, and they are mostly
outdated compared to PyPI. This is the reason why pip should always be used to obtain
new packages in the latest version, as recommended by the Python Packaging Authority
(PyPA). Although it is an independent package, starting from version 2.7.9 and 3.4 of
CPython, it is bundled with every new release by default. Installing the new package is as
simple as this:

.. code-block:: bash

    pip install <package-name>

Among other features, pip allows specific versions of packages to be forced (using the ``pip
install package-name==version`` syntax) and upgraded to the latest version available
(using the ``--upgrade`` switch). The full usage description for most of the command-line
tools presented in the book can be easily obtained simply by running the command with
the ``-h`` or ``--help`` switch, but here is an example session that demonstrates the most
commonly used options:

.. code-block:: bash

    $ pip show pip
    Name: pip
    Version: 18.0
    Summary: The PyPA recommended tool for installing Python packages.
    Home-page: https://pip.pypa.io/
    Author: The pip developers
    Author-email: pypa-dev@groups.google.com
    License: MIT
    Location: /Users/swistakm/.envs/epp-3rd-ed/lib/python3.7/site-packages
    Requires:
    Required-by:

    $ pip install 'pip>=18.0'
    Requirement already satisfied: pip>=18.0 in (...)/lib/python3.7/sitepackages (18.0)

    $ pip install --upgrade pip
    Requirement already up-to-date: pip in (...)/lib/python3.7/site-packages
    (18.0)

In some cases, pip may not be available by default. From Python 3.4 onward (and also
Python 2.7.9), it can always be bootstrapped using the ensurepip module:

.. code-block:: bash

    $ python -m ensurepip
    Looking in links:
    /var/folders/z6/3m2r6jgd04q0m7yq29c6lbzh0000gn/T/tmp784u9bct
    Requirement already satisfied: setuptools in /Users/swistakm/.envs/epp-3rd-ed/lib/python3.7/site-packages (40.4.3)
    Collecting pip
    Installing collected packages: pip
    Successfully installed pip-10.0.1
