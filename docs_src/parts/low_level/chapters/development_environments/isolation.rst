2. Isolating the runtime environment
************************************

``pip`` may be used to install system-wide packages. On UNIX-based and Linux systems, this
will require superuser privileges, so the actual invocation will be as follows:

..  code-block:: bash

    sudo pip install <package-name>

Note that this is not required on Windows since it does not provide the Python interpreter
by default, and Python on Windows is usually installed manually by the user without
superuser privileges.

Installing system-wide packages directly from PyPI is not recommended, and should be
avoided. This may seem like a contradiction to the previous statement that using ``pip`` is a
PyPA recommendation, but there are some serious reasons for that. As we explained
earlier, Python is often an important part of many packages that are available through
operating system package repositories, and may power a lot of important services. System
distribution maintainers put in a lot of effort to select the correct versions of packages to
match various package dependencies. Very often, Python packages that are available from
a system's package repositories contain custom patches, or are purposely kept outdated to
ensure compatibility with some other system components. Forcing an update of such a
package, using pip, to a version that breaks some backward compatibility, might cause
bugs in some crucial system service.

Doing such things on the local computer for development purposes only is also not a good
excuse. Recklessly using pip that way is almost always asking for trouble, and will
eventually lead to issues that are very hard to debug. This does not mean that installing
packages from PyPI is a strictly forbidden thing, but it should be always done
consciously and with an understanding of the related risk.

Fortunately, there is an easy solution to this problem: environment isolation. There are
various tools that allow the isolation of the Python runtime environment at different levels
of system abstraction. The main idea is to isolate project dependencies from packages that
are required by different projects and/or system services. The benefits of this approach are
as follows:

- It solves the Project X depends on version 1.x but, Project Y needs 4.x dilemma. The developer can work on multiple projects with different dependencies that may even collide without the risk of affecting each other.
- The project is no longer constrained by versions of packages that are provided in the developer's system distribution repositories.
- There is no risk of breaking other system services that depend on certain package versions, because new package versions are only available inside such an environment.
- A list of packages that are project dependencies can be easily frozen, so it is very easy to reproduce such an environment on another computer.

If you're working on multiple projects in parallel, you'll quickly find that is impossible to
maintain their dependencies without any kind of isolation.

2.1 Application-level isolation versus system-level isolation
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

The easiest and most lightweight approach to isolation is to use application-level virtual
environments. These focus on isolating the Python interpreter and the packages available
within it. Such environments are very easy to set up, and are very often just enough to
ensure proper isolation during the development of small projects and packages.

Unfortunately, in some cases, this may not be enough to ensure enough consistency and
reproducibility. Despite the fact that software written in Python is usually considered very
portable, it is still very easy to run into issues that occur only on selected systems or even
specific distributions of such systems (for example, Ubuntu versus Gentoo). This is very
common in large and complex projects, especially if they depend on compiled Python
extensions or internal components of the hosting operating system.

In such cases, system-level isolation is a good addition to the workflow. This kind of
approach usually tries to replicate and isolate complete operating systems with all of its
libraries and crucial system components, either with classical system virtualization tools
(for example, VMWare, Parallels, and VirtualBox) or container systems (for example,
Docker and Rocket).


2.2. Python's venv
++++++++++++++++++

There are several ways to isolate Python at runtime. The simplest and most obvious,
although hardest to maintain, is to manually change the ``PATH`` and ``PYTHONPATH``
environment variables and/or move the Python binary to a different, customized place
where we want to store our project's dependencies, in order to affect the way that it
discovers available packages. Fortunately, there are several tools available that can help in
maintaining the virtual environments and packages that are installed for these
environments. These are mainly ``virtualenv`` and ``venv``. What they do under the hood is, in
fact, the same that we would do manually. The actual strategy depends on the specific tool
implementation, but generally they are more convenient to use and can provide additional
benefits.

To create new virtual environment, you can simply use the following command:

.. code-block:: bash

    python3.7 -m venv ENV

Here, ``ENV`` should be replaced by the desired name for the new environment. This will
create a new ``ENV`` directory in the current working directory path. Inside, it will contain a
few new directories:

- ``bin/``: This is where the new Python executable and scripts/executables provided by other packages are stored.
- ``lib/`` and ``include/``: These directories contain the supporting library files for new Python inside the virtual environment. The new packages will be installed in ENV/lib/pythonX.Y/site-packages/.

Once the new environment has been created, it needs to be activated in the current shell
session using UNIX's source command: ``source ENV/bin/activate``

This changes the state of the current shell sessions by affecting its environment variables. In
order to make the user aware that they have activated the virtual environment, it will
change the shell prompt by appending the (``ENV``) string at its beginning. To illustrate this,
here is an example session that creates a new environment and activates it:

.. code-block:: bash

    $ python -m venv example
    $ source example/bin/activate
    (example) $ which python
    /home/swistakm/example/bin/python
    (example) $ deactivate
    $ which python
    /usr/local/bin/python

The important thing to note about ``venv`` is that it depends completely on its state, as stored
on a filesystem. It does not provide any additional abilities to track what packages should
be installed in it. These virtual environments are also not portable, and should not be
moved to another system/machine. This means that the new virtual environment needs to
be created from scratch for each new application deployment. Because of this, there is a
good practice that's used by ``venv`` users to store all project dependencies in
the ``requirements.txt`` file (this is the naming convention), as shown in the following
code:

.. code-block:: python

    # lines followed by hash (#) are treated as a comments
    # strict version names are best for reproducibility
    eventlet==0.17.4
    graceful==0.1.1
    # for projects that are well tested with different
    # dependency versions the relative version specifiers
    # are acceptable too
    falcon>=0.3.0,<0.5.0
    # packages without versions should be avoided unless
    # latest release is always required/desired
    pytz

With such files, all dependencies can be easily installed using ``pip``, because it accepts the
requirements file as its output: ``pip install -r requirements.txt``

What needs to be remembered is that the requirements file is not always the ideal solution,
because it does not define the exact list of dependencies, only those that are to be installed.
So, the whole project can work without problems in some development environments but
will fail to start in others if the requirements file is outdated and does not reflect the actual
state of the environment. There is, of course, the ``pip freeze`` command, which prints all
packages in the current environment, but it should not be used blindly. It will output
everything, even packages that are not used in the project but are installed only for testing.

.. important::

    For Windows users, ``venv`` under Windows uses a different naming
    convention for its internal structure of directories. You need to
    use ``Scripts/``, ``Libs/``, and ``Include/`` instead of ``bin/``, ``lib/``,
    and ``include/``, to better match development conventions on that
    operating system. The commands that are used for activating/deactivating
    the environment are also different; you need to
    use ``ENV/Scripts/activate.bat`` and ``ENV/Scripts/deactivate.bat``
    instead of using source on activate and deactivate scripts.

.. important::

    The Python ``venv`` module provides an additional ``pyvenv`` command-line
    script; since Python 3.6, it has been marked as deprecated and its usage is
    officially discouraged, as the ``pythonX.Y -m venv`` command is explicit
    about what version of Python will be used to create new environments,
    unlike the ``pyvenv`` script.

2.3. System-level environment isolation
+++++++++++++++++++++++++++++++++++++++

In most cases, software implementation can iterate quickly because developers reuse a lot
of existing components. Don't Repeat Yourself: this is a popular rule and motto of many
programmers. Using other packages and modules to include them in the code base is only a
part of that culture. What can also be considered under reused components are binary
libraries, databases, system services, third-party APIs, and so on. Even whole operating
systems should be considered as being reused.

The backend services of web-based applications are a great example of how complex such
applications can be. The simplest software stack usually consists of a few layers (starting
from the lowest):

- A database or other kind of storage
- The application code implemented in Python
- An HTTP server, such as Apache or NGINX

Of course, such stacks can be even simpler, but it is very unlikely. In fact, big applications
are often so complex that it is hard to distinguish single layers. Big applications can use
many different databases, be divided into multiple independent processes, and use many
other system services for caching, queuing, logging, service discovery, and so on. Sadly,
there are no limits for complexity, and it seems that code simply follows the second law of
thermodynamics.

What is really important is that not all software stack elements can be isolated on the level
of Python runtime environments. No matter whether it is an HTTP server, such as Nginx,
or RDBMS, such as PostgreSQL, they are usually available in different versions on different
systems. Making sure that everyone in a development team uses the same versions of every
component is very hard without the proper tools. It is theoretically possible that all
developers in a team working on a single project will be able to get the same versions of
services on their development boxes. But all this effort is futile if they do not use the same
operating system as they do in the production environment. Forcing a programmer to work
on something else rather than their beloved system of choice is impossible.

The problem lies in the fact that portability is still a big challenge. Not all services will work
exactly the same in production environments as they do on the developer's machines, and
this is very unlikely to change. Even Python can behave differently on different systems,
despite how much work is put in to make it cross-platform. Usually, this is well
documented and happens only in places that depend directly on system calls, but relying
on the programmer's ability to remember a long list of compatibility quirks is quite an
error-prone strategy.

A popular solution to this problem is isolating whole systems as an application
environment. This is usually achieved by leveraging different types of system virtualization
tools. Virtualization, of course, reduces performance; but with modern computers that have
hardware support for virtualization, the performance loss is usually negligible. On the
other hand, the list of possible gains is very long:

- The development environment can exactly match the system version and services used in production, which helps to solve compatibility issues
- Definitions for system configuration tools, such as Puppet, Chef, or Ansible (if used), can be reused to configure the development environment
- The newly hired team members can easily hop into the project if the creation of such environments is automated
- The developers can work directly with low-level system features that may not be available on operating systems they use for work, for example, File System in User Space (FUSE), which is not available in Windows.

2.3.1. Virtual development environments using Vagrant
-----------------------------------------------------

Vagrant currently seems to be one of the most popular tools for developers to manage
virtual machines for the purpose of local development. It provides a simple and convenient
way to describe development environments with all system dependencies in a way that
is directly tied to the source code of your project. It is available for Windows, Mac OS, and a
few popular Linux distributions (refer to `https://www.vagrantup.com <https://www.vagrantup.com>`_). It does not have
any additional dependencies. Vagrant creates new development environments in the form
of virtual machines or containers. The exact implementation depends on a choice of
virtualization providers. VirtualBox is the default provider, and it is bundled with the
Vagrant installer, but additional providers are available as well. The most notable choices
are VMware, Docker, Linux Containers (LXC), and Hyper-V.

The most important configuration is provided to Vagrant in a single file
named ``Vagrantfile``. It should be independent for every project. The following are the
most important things it provides:

- Choice of virtualization provider
- A box, which is used as a virtual machine image
- Choice of provisioning method
- Shared storage between the VM and VM's host
- Ports that need to be forwarded between VM and its host

The syntax language for Vagrantfile is Ruby. The example configuration file provides a
good template to start the project and has an excellent documentation, so the knowledge of
this language is not required. Template configuration can be created using a single
command:

.. code-block:: bash

    vagrant init

This will create a new file named ``Vagrantfile`` in the current working directory. The best
place to store this file is usually the root of the related project sources. This file is already a
valid configuration that will create a new VM using the default provider and base box
image. The default ``Vagrantfile`` content that's created with the ``vagrant init`` command
contains a lot of comments that will guide you through the complete configuration process.

The following is a minimal example of ``Vagrantfile`` for the Python 3.7 development
environment based on the Ubuntu operating system, with some sensible defaults that,
among others, enable port 80 forwarding in case you want to do some web development
with Python:

.. code-block:: ruby

    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    Vagrant.configure("2") do |config|
     # Every Vagrant development environment requires a box.
     # You can search for boxes at https://vagrantcloud.com/search.
     # Here we use Bionic version Ubuntu system for x64 architecture.
     config.vm.box = "ubuntu/bionic64"
     # Create a forwarded port mapping which allows access to a specific
     # port within the machine from a port on the host machine and only
     # allow access via 127.0.0.1 to disable public access
     config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
     config.vm.provider "virtualbox" do |vb|
     # Display the VirtualBox GUI when booting the machine
     vb.gui = false
     # Customize the amount of memory on the VM:
     vb.memory = "1024"
     end
     # Enable provisioning with a shell script.
     config.vm.provision "shell", inline: <<-SHELL
     apt-get update
     apt-get install python3.7 -y
     SHELL
    end

In the preceding example, we have set an additional provision of system packages with
simple shell script. When you feel that ``Vagrantfile`` is ready, you can run your virtual
machine using the following command:

.. code-block:: bash

    vagrant up

The initial start can take a few minutes, because the actual box image must be downloaded
from the web. There are also some initialization processes that may take a while every time
the existing VM is brought up, and the amount of time depends on the choice of provider,
image, and your system's performance. Usually, this takes only a couple of seconds. Once
the new Vagrant environment is up and running, developers can connect to it through SSH
using the following shorthand:

.. code-block:: bash

    vagrant ssh

This can be done anywhere in the project source tree below the location of ``Vagrantfile``.
For the developers' convenience, Vagrant will traverse all directories above the user's
current working directory in the filesystem tree, looking for the configuration file and
matching it with the related VM instance. Then, it establishes the secure shell connection, so
the development environment can be interacted with just like an ordinary remote machine.
The only difference is that the whole project source tree (root defined as the location
of Vagrantfile) is available on the VM's filesystem under ``/vagrant/``. This directory is
automatically synchronized with your host filesystem, so you can normally work in the IDE
or editor of your choice run on the host, and can treat the SSH session to your Vagrant VM
just like a normal local Terminal session.

2.3.2. Virtual environments using Docker
----------------------------------------

Containers are an alternative to full machine virtualization. It is a lightweight method of
virtualization, where the kernel and operating system allow multiple isolated user space
instances to be run. OS is shared between containers and the host, so it theoretically
requires less overhead than in full virtualization. Such a container contains only application
code and its system-level dependencies, but, from the perspective of processes running
inside, it looks like a completely isolated system environment.

Software containers got their popularity mostly thanks to Docker, which is one of the
available implementations. Docker allows to describe its container in the form of a simple
text document called ``Dockerfile``. Containers from such definitions can be built and
stored. It also supports incremental changes, so if new things are added to the container
then it does not need to be recreated from scratch.

2.3.2.1. Containerization versus virtualization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Different tools, such as Docker and Vagrant, seem to overlap in features: but the main
difference between them is the reason why these tools were built. Vagrant, as we
mentioned earlier, is built primarily as a tool for development. It allows us to bootstrap the
whole virtual machine with a single command, but does not allow us to simply pack such
an environment as a complete deliverable artifact and deploy or release it. Docker, on the
other hand, is built exactly for that purpose: preparing complete containers that can be
sent and deployed to production as a whole package. If implemented well, this can greatly
improve the process of product deployment. Because of that, using Docker and similar
solutions (Rocket for example) during development only makes more sense if such
containers are also to be used in the deployment process on production.

Due to some implementation nuances, the environments that are based on containers may
sometimes behave differently than environments based on virtual machines. If you decide
to use containers for development, but don't decide to use them on target production
environments, you'll lose some of the consistency guarantees that were the main reason for
environment isolation. But, if you already use containers in your target production
environments, then you should always replicate production conditions rather than using
the same technique. Fortunately, Docker, which is currently the most popular container
solution, provides an amazing ``docker-compose`` tool that makes the management of local
containerized environments extremely easy.

2.3.2.2. Writing your first Dockerfile
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Every Docker-based environment starts with ``Dockerfile``. Dockerfile is a format description
of how to create a Docker image. You can think about the Docker images in a similar way to
how you would think about images of virtual machines. It is a single file (composed of
many layers) that encapsulates all system libraries, files, source code, and other
dependencies that are required to execute your application.

Every layer of a Docker image is described in the Dockerfile by a single instruction in the
following format: ``INSTRUCTION arguments``.

Docker supports plenty of instructions, but the most basic ones that you need to know in
order to get started are as follows:

- ``FROM <image-name>``: This describes the base image that your image will be based on.
- ``COPY <src>... <dst>``: This copies files from the local build context (usually project files) and adds them to the container's filesystem.
- ``ADD <src>... <dst>``: This works similarly to COPY but automatically unpacks archives and allows <src> to be URLs.
- ``RUN <command>``: This runs specified commands on top of previous layers, and commits changes that this command made to the filesystem as a new image layer.
- ``ENTRYPOINT ["<executable>", "<param>", ...]``: This configures the default command to be run as your container. If no entry point is specified anywhere in the image layers, then Docker defaults to /bin/sh -c.
- ``CMD ["<param>", ...]``: This specifies the default parameters for image entry points. Knowing that the default entry point for Docker is /bin/sh -c, this instruction can also take the form of CMD ["<executable>", "<param>", ...], although it is recommended to define the target executable directly in the ENTRYPOINT instruction and use CMD only for default arguments.
- ``WORKDIR <dir>``: This sets the current working directory for any of the following RUN, CMD, ENTRYPOINT, COPY, and ADD instructions.

To properly illustrate the typical structure of ``Dockerfile``, let's assume that we want to
dockerize the built-in Python web server available through the ``http.server`` module with
some predefined static files that this server should serve. The structure of our project files
could be as follows:

.. code-block:: bash

    .
    ├── Dockerfile
    ├── README
    └── static
        ├── index.html
        └── picture.jpg

Locally, you could run that Python's ``http.server`` on a default HTTP port with the
following simple command:

.. code-block:: python

    python3.7 -m http.server --directory static/ 80

This example is of course, very trivial, and using Docker for it is using a sledgehammer to
crack a nut. So, just for the purpose of this example, let's pretend that we have a lot of code
in the project that generates these static files. We would like to deliver only these static files,
and not the code that generates them. Let's also assume that the recipients of our image
know how to use Docker but don't know how to use Python.

So, what we want to achieve is the following:

- Hide some complexity from the user—especially the fact that we use Python and the HTTP server that's built-in into Python
- Package Python3.7 executable with all its dependencies and all static files primarily available in our project directory
- Provide some defaults to run the server on port 80

With all these requirements, our Dockerfile could take the following form:

.. code-block:: docker

    # Let's define base image.
    # "python" is official Python image.
    # The "slim" versions are sensible starting
    # points for other lightweight Python-based images
    FROM python:3.7-slim

    # In order to keep image clean let's switch
    # to selected working directory. "/app/" is
    # commonly used for that purpose.
    WORKDIR /app/

    # These are our static files copied from
    # project source tree to the current working
    # directory.
    COPY static/ static/

    # We would run "python -m http.server" locally
    # so lets make it an entry point.
    ENTRYPOINT ["python3.7", "-m", "http.server"]

    # We want to serve files from static/ directory
    # on port 80 by default so set this as default arguments
    # of the built-in Python HTTP server
    CMD ["--directory", "static/", "80"]

2.3.2.3. Running containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before your container can be started, you'll first need to build an image defined in the
``Dockerfile``. You can build the image using the following command:

.. code-block:: bash

    docker build -t <name> <path>

The ``-t <name>`` argument allows us to name the image with a readable identifier. It is
totally optional, but without it you won't be able to easily reference a newly created image.
The ``<path>`` argument specifies the path to the directory where your Dockerfile is located.
Let's assume that we were already running the command from the root of the project we
presented in the previous section, and we want to tag our image with the name
webserver. The docker build command invocation will be following, and its output
may be as follows:

.. code-block:: bash

    $ docker build -t webserver .

    Sending build context to Docker daemon 4.608kB
    Step 1/5 : FROM python:3.7-slim
    3.7-slim: Pulling from library/python
    802b00ed6f79: Pull complete
    cf9573ca9503: Pull complete
    b2182f7db2fb: Pull complete
    37c0dde21a8c: Pull complete
    a6c85c69b6b4: Pull complete
    Digest:
    sha256:b73537137f740733ef0af985d5d7e5ac5054aadebfa2b6691df5efa793f9fd6d
    Status: Downloaded newer image for python:3.7-slim
     ---> a3aec6c4b7c4
    Step 2/5 : WORKDIR /app/
     ---> Running in 648a5bb2d9ab
    Removing intermediate container 648a5bb2d9ab
     ---> a2489d084377
    Step 3/5 : COPY static/ static/
     ---> 958a04fa5fa8
    Step 4/5 : ENTRYPOINT ["python3.7", "-m", "http.server", "--bind", "80"]
     ---> Running in ec9f2a63c472
    Removing intermediate container ec9f2a63c472
     ---> 991f46cf010a
    Step 5/5 : CMD ["--directory", "static/"]
     ---> Running in 60322d5a9e9e
    Removing intermediate container 60322d5a9e9e
     ---> 40c606a39f7a
    Successfully built 40c606a39f7a
    Successfully tagged webserver:latest

Once created, you can inspect the list of available images using the docker images
command:

.. code-block:: bash

    $ docker images

    REPOSITORY      TAG         IMAGE ID        CREATED         SIZE
    webserver       latest      40c606a39f7a    2 minutes ago   143MB
    python          3.7-slim    a3aec6c4b7c4    2 weeks ago     143MB

.. note::

    The 143 MB of image for a simple Python image may seem like a lot, but it
    isn't really anything to worry about. For the sake of brevity, we have used
    a base image that is simple to use. There are other images that have been
    crafted specially to minimize this size, but these are usually dedicated to
    more experienced Docker users. Also, thanks to the layered structure of
    Docker images, if you're using many containers, the base layers can be
    cached and reused, so an eventual space overhead is rarely an issue.

Once your image is built and tagged, you can run a container using the docker run
command. Our container is an example of a web service, so we will have to additionally tell
Docker that we want to publish the container's ports by binding them locally:

.. code-block:: bash

    docker run -it --rm -p 80:80 webserver

Here is an explanation of the specific arguments of the preceding command:

- ``-it``: These are actually two concatenated options: ``-i`` and ``-t``. ``-i`` (like interactive) keeps STDIN open, even if the container process is detached, and ``-t`` (like tty) allocates pseudo-TTY for the container. In short, thanks to these two options, we will be able to see live logs from ``http.server`` and ensure that the keyboard interrupt will cause the process to exit. It will simply behave the same way as we would start Python, straight from the command line.
- ``--rm``: Tells Docker to automatically remove container when it exits.
- ``-p 80:80``: Tells Docker to publish the container's port 80 by binding port 80 on the host's interface.

2.3.2.4. Setting up complex environments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

While the basic usage of Docker is pretty straightforward for basic setups, it can be bit
overwhelming once you start to use it in multiple projects. It is really easy to forget about
specific command-line options, or which ports should be published on which images. But
things start to be really complicated when you have one service that needs to communicate
with others. Single docker containers should only contain one running process.

This means that you really shouldn't put any additional process supervision tools, such as
Supervisor and Circus, and should instead set up multiple containers that communicate
with each other. Each service may use a completely different image, provide different
configuration options, and expose ports that may or may not overlap.

The best tool that you can use in both simple and complex scenarios is Compose. Compose
is usually distributed with Docker, but in some Linux distributions (for example, Ubuntu),
it may not be available by default, and must be installed as a separate package from the
packages repository. Compose provides a powerful command-line utility named ``docker-compose``,
and allows you to describe multi-container applications using the YAML syntax.

Compose expects the specially named ``docker-compose.yml`` file to be in your project
directory. An example of such a file for our previous project could be as follows:

.. code-block:: yaml

    version: '3'
    services:
         webserver:
             # this tell Compose to build image from
             # local (.) directory
             build: .
             # this is equivalent to "-p" option of
             # the "docker build" command
             ports:
                - "80:80"
             # this is equivalent to "-t" option of
             # the "docker build" command
             tty: true

If you create such a docker-compose.yml file in your project, then your whole application
environment can be started and stopped with two simple commands:

.. code-block:: bash

    docker-compose up
    docker-compose down

2.3.2.5. Reducing the size of containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A common concern of new Docker users is the size of their container images. It's true that
containers provide a lot of space overhead compared to plain Python packages, but it is
usually nothing if we compare the size of images for virtual machines. However, it is still
very common to host many services on a single virtual machine, but with a container-based
approach, you should definitely have a separate image for every service. This means that
with a lot of services, the overhead may become noticeable.

If you want to limit the size of your images, you can use two complementary techniques:

1. Use a base image that is designed specifically for that purpose: Alpine Linux is an example of a compact Linux distribution that is specifically tailored to provide very small and lightweight Docker images. The base image is only 5 MB in size, and provides an elegant package manager that allows you to keep your images compact, too.
2. Take into consideration the characteristics of the Docker overlay filesystem: Docker images consist of layers where each layer encapsulates the difference in the root filesystem between itself and the previous layer. Once the layer is committed the size of the image cannot be reduced. This means that if you need a system package as a build dependency, and it may be later discarded from the image, then instead of using multiple RUN instructions, it may be better to do everything in a single RUN instruction with chained shell commands to avoid excessive layer commits.

These two techniques can be illustrated by the following Dockerfile:

.. code-block:: docker

    # Here we use bare alpine to illustrate
    # package management as it lacks Python
    # by default. For Python projects in general
    # the 'python:3.7-alpine' is probably better
    # choice.
    FROM alpine:3.7

    # Add python3 package as alpine image lacks it by default
    RUN apk add python3

    # Run multiple commands in single RUN instruction
    # so space can be reclaimed after the 'apk del py3-pip'
    # command because image layer is committed only after
    # whole whole instruction.
    RUN apk add py3-pip && \
     pip3 install django && \
     apk del py3-pip

2.3.2.6. Addressing services inside of a Compose environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Complex applications often consist of multiple services that communicate with each other.
Compose allows us to define such applications with ease. The following is an example
``docker-compose.yml`` file that defines the application as a composition of two services:

.. code-block:: yaml

    version: '3'
    services:
        webserver:
            build: .
            ports:
                - "80:80"
            tty: true

        database:
            image: postgres
            restart: always

The preceding configuration defines two services:

- ``webserver``: This is a main application service container with images built from the local Dockerfile
- ``database``: This is a PostgreSQL database container from an official postgres Docker image

We assume that the ``webserver`` service wants to communicate with the ``database`` service
over the network. In order to set up such communications, we need to know the service IP
address or hostname so that it can be used as an application configuration. Thankfully,
Compose is a tool that was designed exactly for such scenarios, so it will make it a lot more
easier for us.

Whenever you start your environment with the ``docker-compose up`` command, Compose
will create a dedicated Docker network by default, and will register all services in that
network using their names as their hostnames. This means that the ``webserver`` service can
use the ``database:5432`` address to communicate with its database (5432 is the default
PostgreSQL port), and any other service in that Compose applicant will be able to access
the HTTP endpoint of the webserver service under the ``http://webserver:80`` address.

Even though the service hostnames in Compose are easily predictable, it isn't good practice
to hardcode any addresses in your application or its configuration. The best approach
would be to provide them as environment variables that can be read by an application on
startup. The following example shows how arbitrary environment variables can be defined
for each service in a ``docker-compose.yml`` file:

.. code-block:: yaml

    version: '3'
    services:
         webserver:
             build: .
             ports:
                - "80:80"
             tty: true
             environment:
                 - DATABASE_HOSTNAME=database
                 - DATABASE_PORT=5432

         database:
             image: postgres
             restart: always

2.3.2.7. Communicating between multiple Compose environments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you build a system composed of multiple independent services and/or applications, you
will very likely want to keep their code in multiple independent code repositories
(projects). The ``docker-compose.yml`` files for every Compose application are usually kept
in the same code repository as the application code. The default network that was created
by Compose for a single application is isolated from the networks of other applications. So,
what can you do if you suddenly want your multiple independent applications to
communicate with each other?

Fortunately, this is another thing that is extremely easy with Compose. The syntax of
the ``docker-compose.yml`` file allows you to define a named external Docker network as
the default network for all services defined in that configuration. The following is an
example configuration that defines an external network named ``my-interservicenetwork``:

.. code-block:: yaml

    version: '3'

    networks:
         default:
            external:
                name: my-interservice-network

    services:
         webserver:
             build: .
             ports:
                - "80:80"
             tty: true
             environment:
                 - DATABASE_HOSTNAME=database
                 - DATABASE_PORT=5432

         database:
             image: postgres
             restart: always

Such external networks are not managed by Compose, so you'll have to create it manually
with the docker network create command, as follows:

.. code-block:: bash

    docker network create my-interservice-network

Once you have done this, you can use this external network in other ``docker-compose.yml``
files for all applications that should have their services registered in the same network. The
following is an example configuration for other applications that will be able to
communicate with both ``database`` and ``webserver`` services over ``my-interservicenetwork``, even though
they are not defined in the same ``docker-compose.yml`` file:

.. code-block:: yaml

    version: '3'
    networks:
         default:
            external:
                name: my-interservice-network
    services:
         other-service:
             build: .
             ports:
                - "80:80"
             tty: true
             environment:
                 - DATABASE_HOSTNAME=database
                 - DATABASE_PORT=5432
                 - WEBSERVER_ADDRESS=http://webserver:80
