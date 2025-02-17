2. Software components
**********************

We have a large system now, and we need to scale it. It also has to be maintainable. At this
point, the concerns aren't only technical but also organizational. This means it's not just
about managing software repositories; each repository will most likely belong to an
application, and it will be maintained by a team who owns that part of the system.

This demands we keep in mind how a large system is divided into different components.
This can have many phases, from a very simple approach about, say, creating Python
packages, to more complex scenarios in a microservice architecture.

The situation could be even more complex when different languages are involved, but in
this chapter, we will assume they are all Python projects.

These components need to interact, as do the teams. The only way this can work at scale is
if all the parts agree on an interface, a contract.

2.1. Packages
+++++++++++++

A Python package is a convenient way to distribute software and reuse code in a more
general way. Packages that have been built can be published to an artifact repository (such
as an internal PyPi server for the company), from where it will be downloaded by the rest
of the applications that require it.

The motivation behind this approach has many elements to it: it's about reusing code at
large, and also achieving conceptual integrity.

Here, we discuss the basics of packaging a Python project that can be published in a
repository. The default repository might be PyPi, but also internal; or
custom setups will work with the same basics.

We are going to simulate that we have created a small library, and we will use that as an
example to review the main points to take into consideration.

Aside from all the open source libraries available, sometimes we might need some extra
functionality: perhaps our application uses a particular idiom repeatedly or relies on a
function or mechanism quite heavily and the team has devised a better function for these
particular needs. In order to work more effectively, we can place this abstraction into a
library, and encourage all team members to use the idioms as provided by it, because doing
so will help avoid mistakes and reduce bugs.

Potentially, there are infinite examples that could suit this scenario. Maybe the application
needs to extract a lot of .tag.gz files (in a particular format) and has faced security
problems in the past with malicious files that ended up with path traversal attacks. As a
mitigation measure, the functionality for abstracting custom file formats securely was put
in a library that wraps the default one and adds some extra checks. This sounds like a good
idea.

Or maybe there is a configuration file that has to be written, or parsed in a particular
format, and this requires many steps to be followed in order; again, creating a helper
function to wrap this, and using it in all the projects that need it, constitutes a good
investment, not only because it saves a lot of code repetition, but also because it makes it
harder to make mistakes.

The gain is not only complying with the DRY principle (avoiding code duplication,
encouraging reuse) but also that the abstracted functionality represents a single point of
reference of how things should be done, hence contributing to the attainment of conceptual
integrity.

In general, the minimum layout for a library would look like this:

.. code-block::

    .
    ├── Makefile
    ├── README.rst
    ├── setup.py
    ├── src
    │    ├── apptool
    │    ├── common.py
    │    ├── __init__.py
    │    └── parse.py
    └── tests
         ├── integration
         └── unit

The important part is the ``setup.py`` file, which contains the definition for the package. In
this file, all the important definitions of the project (its requirements, dependencies, name,
description, and so on) are specified.

The ``apptool`` directory under ``src`` is the name of the library we're working on. This is a
typical Python project, so we place here all the files we need.

An example of the setup.py file could be:

.. code-block:: python

    from setuptools import find_packages, setup


    with open("README.rst", "r") as longdesc:
        long_description = longdesc.read()

    setup(
        name="apptool",
        description="Description of the intention of the package",
        long_description=long_description,
        author="Dev team",
        version="0.1.0",
        packages=find_packages(where="src/"),
        package_dir={"": "src"},
    )

This minimal example contains the key elements of the project. The name argument in the
setup function is used to give the name that the package will have in the repository (under
this name, we run the command to install it, in this case its ``pip install apptool``). It's
not strictly required that it matches the name of the project directory (``src/apptool``), but
it's highly recommended, so its easier for users.

The version is important to keep different releases going on, and then the packages are
specified. By using the ``find_packages()`` function, we automatically discover everything
that's a package, in this case under the ``src/`` directory. Searching under this directory helps
to avoid mixing up files beyond the scope of the project and, for instance, accidentally
releasing tests or a broken structure of the project.

A package is built by running the following commands, assuming its run inside a virtual
environment with the dependencies installed:

.. code-block:: bash

    $VIRTUAL_ENV/bin/pip install -U setuptools wheel
    $VIRTUAL_ENV/bin/python setup.py sdist bdist_wheel

This will place the artifacts in the ``dist/`` directory, from where they can be later published
either to PyPi or to the internal package repository of the company.

The key points in packaging a Python project are:

- Test and verify that the installation is platform-independent and that it doesn't rely on any local setup (this can be achieved by placing the source files under an ``src/`` directory)
- Make sure that unit tests aren't shipped as part of the package being built
- Separate dependencies: what the project strictly needs to run is not the same as what developers require
- It's a good idea to create entry points for the commands that are going to be required the most

The ``setup.py`` file supports multiple other parameters and configurations and can be
effected in a much more complicated manner. If our package requires several operating
system libraries to be installed, it's a good idea to write some logic in the ``setup.py`` file to
compile and build the extensions that are required. This way, if something is amiss, it will
fail early on in the installation process, and if the package provides a helpful error message,
the user will be able to fix the dependencies more quickly and continue.

Installing such dependencies represents another difficult step in making the application
ubiquitous, and easy to run by any developer regardless of their platform of choice. The
best way to surmount this obstacle is to abstract the platform by creating a Docker image.

2.2. Containers
+++++++++++++++

This chapter is dedicated to architecture, so the term container refers to something
completely different from a Python container (an object with a ``__contains__`` method).
A container is a process that runs in the operating
system under a group with certain restrictions and isolation considerations. Concretely we
refer to Docker containers, which allow managing applications (services or processes) as
independent components.

Containers represent another way of delivering software. Creating Python packages taking
into account the considerations in the previous section is more suitable for libraries, or
frameworks, where the goal is to reuse code and take advantage of using a single place
where specific logic is gathered.

In the case of containers, the objective will not be creating libraries but applications (most of
the time). However, an application or platform does not necessarily mean an entire service.
The idea of building containers is to create small components that represent a service with a
small and clear purpose.

In this section, we will mention Docker when we talk about containers, and we will explore
the basics of how to create Docker images and containers for Python projects. Keep in mind
that this is not the only technology for launching applications into containers, and also that
it's completely independent of Python.

A Docker container needs an image to run on, and this image is created from other base
images. But the images we create can themselves serve as base images for other containers.
We will want to do that in cases where there is a common base in our application that can
be shared across many containers. A potential use would be creating a base image that
installs a package (or many) in the way we described in the previous section, and also all of
its dependencies, including those at the operating system level. A package we create can depend not only on other Python
libraries, but also on a particular platform (a specific operating system), and particular
libraries preinstalled in that operating system, without which the package will simply not
install and will fail.

Containers are a great portability tool for this. They can help us ensure that our application
will have a canonical way of running, and it will also ease the development process a lot
(reproducing scenarios across environments, replicating tests, on-boarding new team
members, and so on).

As packages are the way we reuse code and unify criteria, containers represent the way we
create the different services of the application. They meet the criteria behind the principle of
separation of concerns (SoC) of the architecture. Each service is another kind of component
that will encapsulate a set of functionalities independently of the rest of the application.
These containers ought to be designed in such a way that they favor maintainability: if the
responsibilities are clearly divided, a change in a service should not impact any other part
of the application whatsoever.
