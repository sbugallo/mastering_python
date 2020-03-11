2. Approaches to deployment automation
**************************************

With the advent of application containerization (Docker and similar technologies), modern
software provisioning tools (for example, Puppet, Chef, Ansible, and Salt),
and infrastructure management systems (for example, Terraform and SaltStack)
development and operations teams have a variety of ways in which they can organize and
manage their code deployments and configuration of remote systems. Each solution has
pros and cons, so advanced automation tools should be chosen very wisely with respect to
the favored development processes and methodologies.

Fast paced teams that use microservice architecture and deploy code often (maybe even
simultaneously in parallel versions) will definitely favor container orchestration systems
such as Kubernetes or use dedicated services provided by their cloud vendor (for example,
AWS). Teams that build old-style big monolithic applications and run them on their own
bare-metal servers might want to use more low-level automation and software provisioning
systems. Actually, there is no rule and you can find teams of every size using every possible
approach to software provisioning, code deployments, and application orchestration. The
limiting factors here are resources and knowledge.

That's why it's really hard to briefly provide a set of common tools and solutions that
would fit the needs and capabilities of every developer and every team. Because of that, in
this chapter, we will focus only on a pretty simple approach to automation using Fabric.
We could say that this is outdated. And that's probably true. What seem to be the most
modern are container orchestrations systems in the style of Kubernetes that allow you to
leverage Docker containers for fast, maintainable, scalable, and reproducible environments.
But these systems have quite a steep learning curve and it's impossible to introduce them in
just a few sections of a single chapter. Fabric, on the other hand, is very simple and easy to
grasp so it is a really great tool to introduce someone to the concept of automation.

2.1. Using Fabric for deployment automation
+++++++++++++++++++++++++++++++++++++++++++

For very small projects, it may be possible to deploy your code by hand, that is, by manually
typing the sequence of commands through the remote shells that are necessary to install a
new version of code and execute it on a remote shell. Anyway, even for an average-sized
project, this is error-prone and tedious and should be considered a waste of most of the
precious resource you have, your own time.

The solution for that is automation. The simple thumb rule could be the following:

*"If you needed to perform the same task manually at least twice, you should automate it so you won't need to do it for the third time."*

There are various tools that allow you to automate different things, including the following:

- Remote execution tools such as Fabric are used for on-demand automated execution of code on multiple remote hosts.
- Configuration management tools such as Chef, Puppet, CFEngine, Salt, and Ansible are designed for automatized configuration of remote hosts (execution environments). They can be used to set up backing services (databases, caches, and so on), system permissions, users, and so on. Most of them can be used also as a tool for remote execution (such as Fabric) but, depending on their architecture, this may be more or less convenient.

Configuration management solutions is a complex topic. The
truth is that the simplest remote execution frameworks have the lowest entry barrier and
are the most popular choice, at least for small projects. In fact, every configuration
management tool that provides a way to declaratively specify configuration of your
machines has a remote execution layer implemented somewhere deep inside.

Also, some configuration management tools may not be best suited for actual automated
code deployment. One such example is Puppet, which really discourages the explicit
running of any shell commands. This is why many people choose to use both types of
solution to complement each other: configuration management for setting up a system-level
environment and on-demand remote execution for application deployment.

Fabric (`http://www.fabfile.org/ <http://www.fabfile.org/>`_ ) is so far the most popular solution used by
Python developers to automate remote execution. It is a Python library and command-line tool for
streamlining the use of SSH for application deployment or systems administration tasks.
We will focus on it because it is relatively easy to start with. Keep in mind that, depending
on your needs, it may not be the best solution to your problems. Anyway, it is a great
example of a utility that can add some automation to your operations, if you don't have any
yet.

You could, of course, automate all of the work using only Bash scripts but this is very
tedious and error-prone. Python has more convenient ways for string processing and
encourages code modularization. Fabric is in fact only a tool for gluing the execution of
commands via SSH. It means that you still need to know how to use the command-line
interface and its utilities in your remote environment.

So, if you want to strictly follow the Twelve-Factor App methodology, you should not
maintain its code in the source tree of the deployed application.

Complex projects are, in fact, very often built from various components maintained as
separate code bases, so this is another reason why it is a good approach to have one
separate repository for all of the project component configurations and Fabric scripts. This
makes deployment of different services more consistent and encourages good code reuse.

To start working with Fabric, you need to install the ``fabric`` package (using ``pip``) and
create a script named ``fabfile.py``. That script is usually located in the root of your project.
Note that ``fabfile.py`` can be considered a part of your project configuration.

But before we create our ``fabfile`` let's define some initial utilities that will help us to set up
the project remotely. Here's a module that we will call ``fabutils``:

.. code-block:: python

    import os


    # Let's assume we have private package repository created
    # using 'devpi' project
    PYPI_URL = 'http://devpi.webxample.example.com'

    # This is arbitrary location for storing installed releases.
    # Each release is a separate virtual environment directory
    # which is named after project version. There is also a
    # symbolic link 'current' that points to recently deployed
    # version. This symlink is an actual path that will be used
    # for configuring the process supervision tool for example:
    # .
    # ├── 0.0.1
    # ├── 0.0.2
    # ├── 0.0.3
    # ├── 0.1.0
    # └── current -> 0.1.0/

    REMOTE_PROJECT_LOCATION = "/var/projects/webxample"
    def prepare_release(c):
        """ Prepare a new release by creating source distribution and
        uploading to out private package repository
        """
        c.local(f'python setup.py build sdist')
        c.local(f'twine upload --repository-url {PYPI_URL}')

    def get_version(c):
        """ Get current project version from setuptools """
        return c.local('python setup.py --version').stdout.strip()

    def switch_versions(c, version):
        """ Switch versions by replacing symlinks atomically """
        new_version_path = os.path.join(REMOTE_PROJECT_LOCATION, version)
        temporary = os.path.join(REMOTE_PROJECT_LOCATION, 'next')
        desired = os.path.join(REMOTE_PROJECT_LOCATION, 'current')

        # force symlink (-f) since probably there is a one already
        c.run(f"ln -fsT {new_version_path} {temporary}")
        # mv -T ensures atomicity of this operation
        c.run(f"mv -Tf {temporary} {desired}" )

An example of a final fabfile that defines a simple deployment procedure will look like
this:

.. code-block:: python

    from fabric import task
    from .fabutils import *


    @task
    def uptime(c):
        """
        Run uptime command on remote host - for testing connection.
        """
        c.run("uptime")

    @task
    def deploy(c):
        """ Deploy application with packaging in mind """
        version = get_version(c)

        pip_path = os.path.join(
            REMOTE_PROJECT_LOCATION, version, 'bin', 'pip'
        )

        if not c.run(f"test -d {REMOTE_PROJECT_LOCATION}", warn=True):
            # it may not exist for initial deployment on fresh host
            c.run(f"mkdir -p {REMOTE_PROJECT_LOCATION}")

        with c.cd(REMOTE_PROJECT_LOCATION):
            # create new virtual environment using venv
            c.run(f'python3 -m venv {version}')
            c.run(f"{pip_path} install webxample=={version} --index-url {PYPI_URL}")

        switch_versions(c, version)
        # let's assume that Circus is our process supervision tool
        # of choice.
        c.run('circusctl restart webxample')

Every function decorated with ``@task`` is now treated as an available subcommand to
the fab utility provided with the fabric package. You can list all of the available
subcommands using the ``-l`` or ``--list`` switch. The code is shown in the following snippet:

.. code-block:: bash

    $ fab --list
    Available commands:
        deploy Deploy application with packaging in mind
        uptime Run uptime command on remote host - for testing connection.

Now, you can deploy the application to the given environment type with only the
following single shell command:

.. code-block:: bash

    $ fab -H myhost.example.com deploy

Note that the preceding ``fabfile`` serves only illustrative purposes. In your own code, you
might want to provide extensive failure handling and try to reload the application without
the need to restart the web worker process. Also, some of the techniques presented here
may not be obvious right now but will be explained later in this chapter. These include the
following:

- Deploying an application using the private package repository
- Using Circus for process supervision on the remote host