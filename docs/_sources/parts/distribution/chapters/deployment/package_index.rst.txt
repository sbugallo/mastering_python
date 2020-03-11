3. Index mirroring
******************

These are three main reasons why you might want to run your own index of Python
packages:

- The official Python Package Index does not have any availability guarantees. It is run by Python Software Foundation thanks to numerous donations. Because of that, it means that this site can be down at the most inconvenient time. You don't want to stop your deployment or packaging process in the middle due to PyPI outage.
- It is useful to have reusable components written in Python properly packaged, even for the closed source that will never be published publicly. It simplifies code base because packages that are used across the company for different projects do not need to be vendored. You can simply install them from the repository. This simplifies maintenance for such shared code and might reduce development costs for the whole company if it has many teams working on different projects.
- It is a very good practice to have your entire project packaged using ``setuptools``. Then, deployment of the new application version is often as simple as running ``pip install --update my-application``.

.. tip::
    **Code vendoring** is a practice of including sources of the external package
    in the source code (repository) of other projects. It is usually done when
    the project's code depends on a specific version of some external package
    that may also be required by other packages (and in a completely different
    version).

    For instance, the popular ``requests`` package uses some version
    of ``urllib3`` in its source tree because it is very tightly coupled to it and is
    very unlikely to work with any other version of urllib3 . An example of
    a module that is particularly often used by others is ``six``. It can be found
    in sources of numerous popular projects such as Django
    (``django.utils.six``), Boto (``boto.vendored.six``), or Matplotlib
    (``matplotlib.externals.six``).

    Although vendoring is practiced even by some large and successful open
    source projects, it should be avoided if possible. This has justifiable usage
    only in certain circumstances and should not be treated as a substitute of
    package dependency management.

3.1. PyPI mirroring
+++++++++++++++++++

The problem of PyPI outages can be somehow mitigated by allowing the installation tools
to download packages from one of its mirrors. In fact, the official Python Package Index is
already served through **Content Delivery Network (CDN)**, so it is intrinsically mirrored.
This does not change the fact that it seems to have some bad days from time to time. Using
unofficial mirrors is not a solution here because it might raise some security concerns.

The best solution is to have your own PyPI mirror that will have all of the packages you
need. The only party that will use it is you, so it will be much easier to ensure proper
availability. The other advantage is that whenever this service goes down, you don't need
to rely on someone else to bring it up. The mirroring tool maintained and recommended by
PyPA is **bandersnatch**
(`https://pypi.python.org/pypi/bandersnatch <https://pypi.python.org/pypi/bandersnatch>`_). It allows you to
mirror the whole content of Python Package Index and it can be provided as the ``index-url``
option for the repository section in the ``.pypirc`` file (as explained in the previous
chapter). This mirror does not accept uploads and does not have the web part of PyPI.
Anyway, beware! A full mirror might require hundreds of gigabytes of storage and its size
will continue to grow over time.

But why stop at a simple mirror while we have a much better alternative? There is a very
low chance that you will require a mirror of the whole package index. Even with a project
that has hundreds of dependencies, it will be only a minor fraction of all of the available
packages. Also, not being able to upload your own private package is a huge limitation of
such a simple mirror. It seems that the added value of using bandersnatch is very low for
such a high price. And this is true in most situations. If the package mirror is to be
maintained only for single or a few projects, a much better approach is to
use **devpi** (`http://doc.devpi.net/ <http://doc.devpi.net/>`_). It is a PyPI-compatible package index
implementation that provides both of the following:

- A private index to upload nonpublic packages
- Index mirroring

The main advantage of devpi over bandersnatch is how it handles mirroring. It can, of
course, do a full general mirror of other indexes like bandersnatch does, but it is not its
default behavior. Instead of doing a rather expensive backup of the whole repository, it
maintains mirrors for packages that were already requested by clients. So, whenever a
package is requested by the installation tool (``pip``, ``setuptools``, and ``easy_install``), if it
does not exist in the local mirror, the devpi server will attempt to download it from the
mirrored index (usually PyPI) and serve. Once the package is downloaded, the devpi will
periodically check for its updates to maintain a fresh state of its mirror.

The mirroring approach leaves a slight risk of failure when you request a new package that
was not yet mirrored when the upstream package index has an outage. Anyway, this risk is
reduced, thanks to the fact that in most deployments you will depend only on packages
that were already mirrored in the index. The mirror state for packages that were already
requested has eventual consistency guarantee and new versions will be downloaded
automatically. This seems to be a very reasonable trade off.

Now let's see how to properly bundle and build additional non-Python resources in your
Python application.

3.2. Bundling additional resources with your Python package
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Modern web applications have a lot of dependencies and often require a lot of steps to
properly install on the remote host. For instance, the typical bootstrapping process for a
new version of the application on a remote host consists of the following steps:

1. Create a new virtual environment for isolation.
2. Move the project code to the execution environment.
3. Install the latest project requirements (usually from the ``requirements.txt`` file).
4. Synchronize or migrate the database schema.
5. Collect static files from project sources and external packages to the desired location.
6. Compile localization files for applications available in different languages.

For more complex sites, there might be lot of additional tasks mostly related to frontend
code that is independent from previously defined tasks, as in the following example:

1. Generate CSS files using preprocessors such as SASS or LESS.
2. Perform minification, obfuscation, and/or concatenation of static files (JavaScript and CSS files).
3. Compile code written in JavaScript superset languages (CoffeeScript, TypeScript, and so on) to native JS.
4. Preprocess response template files (minification, style inlining, and so on).

Nowadays, for these kind of applications that require a lot of additional assets to be
prepared, most developers would probably use Docker images. Dockerfiles allow you to
easily define all of the steps that are necessary to bundle all assets with your application
image. But if you don't use Docker, it means that all of these steps must be automated using
other tools such as Make, Bash, Fabric, or Ansible. Still, it is not a good idea to do all of
these steps directly on the remote hosts where the application is being installed. Here are
the reasons:

- Some of the popular tools for processing static assets can be either CPU or memory intensive. Running them in production environments can destabilize your application execution.
- These tools very often will require additional system dependencies that may not be required for the normal operation of your projects. These are mostly additional runtime environments such as JVM, Node, or Ruby. This adds complexity to configuration management and increases the overall maintenance costs.
- If you are deploying your application to multiple servers (tens, hundreds, or thousands), you are simply repeating a lot of work that could be done once. If you have your own infrastructure, then you may not experience the huge increase of costs, especially if you perform deployments in periods of low traffic. But if you run cloud computing services in the pricing model that charges you extra for spikes in load or generally for execution time, then this additional cost may be substantial on a proper scale.
- Most of these steps just take a lot of time. You are installing your code on remote servers, so the last thing you want is to have your connection interrupted by some network issue. By keeping the deployment process quick, you are lowering the chance of deployment interruption.

Obviously, the results of these predeployment steps can't be included in your application
code repository either. Simply, there are things that must be done with every release and
you can't change that. It is obviously a place for proper automation but the clue is to do it in
the right place and at the right time.

Most of the things, such as static collection and code/asset preprocessing, can be done
locally or in a dedicated environment, so the actual code that is deployed to the remote
server requires only a minimal amount of on-site processing. The following are the most
notable of such deployment steps, either in the process of building distribution or installing
a package:

1. Installation of Python dependencies and transferring of static assets (CSS files and JavaScript) to the desired location can be handled as a part of the ``install`` command of the ``setup.py`` script.
2. Preprocessing of code (processing JavaScript supersets, minification/obfuscation/concatenation of assets, and running SASS or LESS) and things such as localized text compilation (for example, ``compilemessages`` in Django) can be a part of the ``sdist``/``bdist`` command of the ``setup.py`` script.

Inclusion of preprocessed code other than Python can be easily handled with the
proper ``MANIFEST.in`` file. Dependencies are, of course, best provided as
an ``install_requires`` argument of the ``setup()`` function call from
the setuptools package.

Packaging the whole application, of course, will require some additional work from you,
such as providing your own custom ``setuptools`` commands or overriding the existing
ones, but it gives you a lot of advantages and makes project deployment a lot faster and
reliable.

Let's use a Django-based project (in Django 1.9 version) as an example. I have chosen this
framework because it seems to be the most popular Python project of this type, so there is a
high chance that you already know it a bit. A typical structure of files in such a project
might look like the following:

.. code-block:: bash

    $ tree . -I __pycache__ --dirsfirst
    .
    ├── webxample
    │    ├── conf
    │    │    ├── __init__.py
    │    │    ├── settings.py
    │    │    ├── urls.py
    │    │    └── wsgi.py
    │    ├── locale
    │    │    ├── de
    │    │    │    └── LC_MESSAGES
    │    │    │         └── django.po
    │    │    ├── en
    │    │    │    └── LC_MESSAGES
    │    │    │         └── django.po
    │    │    └── pl
    │    │         └── LC_MESSAGES
    │    │              └── django.po
    │    ├── myapp
    │    │    ├── migrations
    │    │    │    └── __init__.py
    │    │    ├── static
    │    │    │    ├── js
    │    │    │    │    └── myapp.js
    │    │    │    └── sass
    │    │    │         └── myapp.scss
    │    │    ├── templates
    │    │    │    ├── index.html
    │    │    │    └── some_view.html
    │    │    ├── __init__.py
    │    │    ├── admin.py
    │    │    ├── apps.py
    │    │    ├── models.py
    │    │    ├── tests.py
    │    │    └── views.py
    │    ├── __init__.py
    │    └── manage.py
    ├── MANIFEST.in
    ├── README.md
    └── setup.py
    15 directories, 23 files

Note that this slightly differs from the usual Django project template. By default, the name
of the package that contains the WSGI application, the settings module, and the URL
configuration has the same name as the project. Because we decided to take the packaging
approach, this would be named as ``webxample``. This can cause some confusion, so it is
better to rename it to ``conf``. Without digging into the possible implementation details, let's
just make the following few simple assumptions:

- Our example application has some external dependencies. Here, it will be two popular Django packages: ``djangorestframework`` and ``django-allauth``, plus one non-Django package: ``gunicorn``.
- ``djangorestframework`` and ``django-allauth`` are provided as ``INSTALLED_APPS`` in the ``webexample.webexample.settings`` module.
- The application is localized in three languages (German, English, and Polish) but we don't want to store the compiled ``gettext`` messages in the repository.
- We are tired of vanilla CSS syntax, so we decided to use a more powerful SCSS language that we translate into CSS using SASS.

Knowing the structure of the project, we can write our ``setup.py`` script in a way that
makes ``setuptools`` handle the following:

- Compilation of SCSS files under ``webxample/myapp/static/scss``
- Compilation of ``gettext`` messages under ``webexample/locale`` from ``.po`` to ``.mo`` format
- Installation of the requirements
- A new script that provides an entry point to the package, so we will have the custom command instead of the ``manage.py`` script

We have a bit of luck here: Python binding for ``libsass``, a C/C++ port of the SASS engine,
provides some integration with ``setuptools`` and ``distutils``. With only a little
configuration, it provides a custom setup.py command for running the SASS compilation.
This is shown in the following code:

.. code-block:: python

    from setuptools import setup
    setup(
        name='webxample',
        setup_requires=['libsass == 0.6.0'],
        sass_manifests={
        'webxample.myapp': ('static/sass', 'static/css')
        }
    )

So, instead of running the ``sass`` command manually or executing a subprocess in
the ``setup.py`` script, we can type python ``setup.py build_scss`` and have our SCSS files
compiled to CSS. This is still not enough. It makes our life a bit easier but we want the
whole distribution fully automated so there is only one step for creating new releases. To
achieve this goal, we are forced to override some of the existing ``setuptools`` distribution
commands.

The example ``setup.py`` file that handles some of the project preparation steps through
packaging might look like this:

.. code-block:: python

    import os
    from setuptools import setup
    from setuptools import find_packages
    from distutils.cmd import Command
    from distutils.command.build import build as _build

    try:
        from django.core.management.commands.compilemessages \
        import Command as CompileCommand
    except ImportError:
        # note: during installation django may not be available
        CompileCommand = None
        # this environment is requires


    os.environ.setdefault(
        "DJANGO_SETTINGS_MODULE", "webxample.conf.settings"
    )


    class build_messages(Command):
        """ Custom command for building gettext messages in Django """
        description = """compile gettext messages"""
        user_options = []

        def initialize_options(self):
            pass

        def finalize_options(self):
            pass

        def run(self):
            if CompileCommand:
                CompileCommand().handle(
                    verbosity=2, locales=[], exclude=[]
                )
            else:
                raise RuntimeError("could not build translations")


    class build(_build):
        """ Overridden build command that adds additional build steps """
        sub_commands = [
            ('build_messages', None),
            ('build_sass', None),
        ] + _build.sub_commands

        setup(
            name='webxample',
            setup_requires=[
                'libsass == 0.6.0',
                'django == 1.9.2'
            ],
            install_requires=[
                'django == 1.9.2',
                'gunicorn == 19.4.5',
                'djangorestframework == 3.3.2',
                'django-allauth == 0.24.1'
            ],
            packages=find_packages('.'),
            sass_manifests={
            '   webxample.myapp': ('static/sass', 'static/css')
            },
            cmdclass={
                'build_messages': build_messages,
                'build': build
            },
            entry_points={
                'console_scripts': {
                    'webxample = webxample.manage:main'
                }
            }
        )

With such an implementation, we can build all assets and create the source distribution of a
package for the ``webxample`` project using the following single Terminal command:

.. code-block:: bash

    $ python setup.py build sdist

If you already have your own package index (created with devpi), you can add
the ``install`` subcommand or use ``twine`` so this package will be available for installation
with ``pip`` in your organization. If we look into a structure of source distribution created
with our ``setup.py`` script, we can see that it contains the following
compiled ``gettext`` messages and CSS style sheets generated from SCSS files:

.. code-block:: bash

    $ tar -xvzf dist/webxample-0.0.0.tar.gz 2> /dev/null
    $ tree webxample-0.0.0/ -I __pycache__ --dirsfirst
    webxample-0.0.0/
    ├── webxample
    │    ├── conf
    │    │    ├── __init__.py
    │    │    ├── settings.py
    │    │    ├── urls.py
    │    │    └── wsgi.py
    │    ├── locale
    │    │    ├── de
    │    │    │    └── LC_MESSAGES
    │    │    │         ├── django.mo
    │    │    │         └── django.po
    │    │    ├── en
    │    │    │    └── LC_MESSAGES
    │    │    │         ├── django.mo
    │    │    │         └── django.po
    │    │    └── pl
    │    │         └── LC_MESSAGES
    │    │              ├── django.mo
    │    │              └── django.po
    │    ├── myapp
    │    │    ├── migrations
    │    │    │    └── __init__.py
    │    │    ├── static
    │    │    │    ├── js
    │    │    │    │    └── myapp.js
    │    │    │    └── sass
    │    │    │         └── myapp.scss.css
    │    │    ├── templates
    │    │    │    ├── index.html
    │    │    │    └── some_view.html
    │    │    ├── __init__.py
    │    │    ├── admin.py
    │    │    ├── apps.py
    │    │    ├── models.py
    │    │    ├── tests.py
    │    │    └── views.py
    │    ├── __init__.py
    │    └── manage.py
    ├── webxample.egg-info
    │    ├── PKG-INFO
    │    ├── SOURCES.txt
    │    ├── dependency_links.txt
    │    ├── requires.txt
    │    └── top_level.txt
    ├── MANIFEST.in
    ├── README.md
    └── setup.py
    16 directories, 33 files

The additional benefit of using this approach is that we were able to provide our own entry
point for the project in place of Django's default ``manage.py`` script. Now, we can run any
Django management command using this entry point, for instance:

.. code-block:: bash

    $ webxample migrate
    $ webxample collectstatic
    $ webxample runserver

This required a little change in the ``manage.py`` script for compatibility with
the ``entry_points`` argument in ``setup()``, so the main part of its code is wrapped with
the ``main()`` function call. This is shown in the following code:

.. code-block:: python

    #!/usr/bin/env python3
    import os
    import sys


    def main():
        os.environ.setdefault(
            "DJANGO_SETTINGS_MODULE", "webxample.conf.settings"
        )

        from django.core.management import execute_from_command_line
        execute_from_command_line(sys.argv)

    if __name__ == "__main__":
        main()

Unfortunately, a lot of frameworks (including Django) are not designed with the idea of
packaging your projects that way in mind. It means that, depending on the advancement of
your application, converting it to a package may require a lot of changes. In Django, this
often means rewriting many of the implicit imports and updating a lot of configuration
variables in your settings file.

The other problem here is consistency of releases created using Python packaging. If
different team members are authorized to create application distribution, it is crucial that
this process takes place in the same replicable environment. Especially when you do a lot of
asset preprocessing, it is possible that the package created in two different environments
will not look the same, even if it is created from the same code base. This may be due to
different versions of tools used during the build process. The best practice is to move the
distribution responsibility to some continuous integration/delivery system such as Jenkins,
Buildbot, Travis CI, or similar. The additional advantage is that you can assert that the
package passes all of the required tests before going to distribution. You can even make the
automated deployment as a part of such a continuous delivery system.

Mind that although distributing your code as Python packages using ``setuptools`` might
seem elegant, it is actually not simple and effortless. It has potential to greatly simplify your
deployments and so it is definitely worth trying but it comes with the cost of increased
complexity. If your preprocessing pipeline for your application grows too complex, you
should definitely consider building Docker images and deploying your application as
containers.

Deployment with Docker requires some additional setup and orchestration but in the long
term saves a lot of time and resources that are otherwise required to maintain repeatable
build environments and complex preprocessing pipelines.
