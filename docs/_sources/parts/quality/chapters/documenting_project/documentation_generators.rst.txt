3. Popular documentation generators
***********************************

As stated previously, software documentation may have varied readership. Accessing
documentation directly from project source code is often natural to users that are
programmers developing a given project. But this way of accessing project documentation
may not be the most convenient for others. Also, some companies may have requirements
to deliver documentation to their clients in a printable form.

This is why documentation generation tools are so important. They allow you to benefit
from documentation being treated as code while still maintaining the ability to have a
deliverable document that can be browsed, searched, and read without access to the
original source code. The Python ecosystem comes with a variety of amazing open source
tools that allow you to generate project documentation directly from your source code. The
two most popular tools in the Python community for generating user-friendly
documentations are Sphinx and MkDocs. We will discuss them briefly in the following
sections.

3.1. Sphinx
+++++++++++

Sphinx (`<http://sphinx.pocoo.org>`_) is a set of scripts and ``docutils`` extensions that can be
used to generate an HTML structure from the tree of plain text documents that are created
using the reStructuredText syntax language. Sphinx also supports multiple other
documentation output formats, like man pages, PDF, or even LaTex. This tool is used (for
instance) to build official Python documentation and is very popular among many open
source Python projects. It provides a really nice browsing system, together with a light but
sufficient client-side JavaScript search engine. It also uses ``pygments`` for rendering code
examples, which produces really nice syntax highlights.

Sphinx can be easily configured to stick with the document landscape we defined in the
previous section. It can be easily installed with ``pip`` as a ``Sphinx`` package.

The easiest way to start working with Sphinx is to use the ``sphinx-quickstart`` script. This
utility will generate a script together with ``Makefile``, which can be used to generate the
web documentation every time it is needed. It will interactively ask you some questions
and then bootstrap the whole initial documentation source tree and configuration file. Once
it is done, you can easily tweak it whenever you want. Let's assume we have already
bootstrapped the whole Sphinx environment and we want to see its HTML representation.
This can be easily done using the ``make html`` command, as follows:

.. code-block:: bash

    project/docs $ make html
    sphinx-build -b html -d _build/doctrees
    . _build/html
    Running Sphinx v1.3.6
    making output directory...
    loading pickled environment... not yet created
    building [mo]: targets for 0 po files that are out of date
    building [html]: targets for 1 source files that are out of date
    updating environment: 1 added, 0 changed, 0 removed
    reading sources... [100%] index
    looking for now-outdated files... none found
    pickling environment... done
    checking consistency... done
    preparing documents... done
    writing output... [100%] index
    generating indices... genindex
    writing additional pages... search
    copying static files... done
    copying extra files... done
    dumping search index in English (code: en) ... done
    dumping object inventory... done
    build succeeded.
    Build finished. The HTML pages are in _build/html.

Besides the HTML versions of the documents, the tool also builds automatic pages, such as
a module list and an index. Sphinx provides a few ``docutils`` extensions to drive these
features. These are the main ones:

- A directive that builds a table of contents
- A marker that can be used to register a document as a module helper
- A marker to add an element in the index

3.1.1. Working with the index pages
-----------------------------------

Sphinx provides a ``toctree`` directive that can be used to inject a table of contents in a
document, with links to other documents. Each line must be a file with its relative path,
starting from the current document. Glob-style names can also be provided to add several
files that match the expression.

For example, the index file in the ``cookbook`` folder, which we previously defined in the
producer's landscape, can look like this:

.. code-block:: rst

    ========
    Cookbook
    ========

    Welcome to the Cookbook.

    Available recipes:

    .. toctree::
       :glob:
       *

With this syntax, the HTML page will display a list of all the reStructuredText documents
available in the ``cookbook`` folder. This directive can be used in all the index files to build
browsable documentation.

3.1.2. Registering module helpers
---------------------------------

For module helpers, a marker can be added so that it is automatically listed and available in
the module's index page, as follows:

.. code-block:: rst

    =======
    session
    =======

    .. module:: db.session

    The module session...

Notice that the ``db`` prefix here can be used to avoid module collision. Sphinx will use it as a
module category and will group all modules that start with ``db.`` in this category.

3.1.3. Adding index markers
---------------------------

Another option can be used to fill the index page by linking the document to an entry, as
follows:

.. code-block:: rst

    =======
    session
    =======

    .. module:: db.session

    .. index::
       Database Access
       Session

    The module session...

Two new entries, ``Database Access`` and ``Session``, will be added in the index page.

3.1.4. Cross-references
-----------------------

Finally, Sphinx provides an inline markup to set cross-references. For instance, a link to a
module can be done like this:

.. code-block:: rst

    :mod:`db.session`

Here, ``:mod:`` is the module marker's prefix and ``db.session`` is the name of the module
to be linked to (as registered previously). Keep in mind that ``:mod:``, as well as the previous
elements, are the specific directives that were introduced in reStructuredText by Sphinx.

.. important::

    Sphinx provides a lot more features that you can discover on its website.
    For instance, the ``autodoc`` feature is a great option to automatically extract
    your doctests to build the documentation. For more information, refer
    to `<http://sphinx.pocoo.org>`_.

3.2. MkDocs
+++++++++++

MkDocs (`<https://www.mkdocs.org/>`_) is a very minimalistic static page generator that can
be used to document your projects. It lacks built-in ``autodoc`` features, similar to those in
Sphinx, but uses the lot simpler and readable Markdown markup language. It is also really
extensible. It is definitely easier to write a MkDocs plugin than a docutils extension that
could be used by Sphinx. So, if you have very specific documentation needs that cannot be
satisfied by existing tools and their extensions are available at the moment, then MkDocs
provides a very good foundation for building something custom-tailored.

3.3. Documentation building and continuous integration
++++++++++++++++++++++++++++++++++++++++++++++++++++++

Sphinx and similar documentation generation tools really improve the readability and
experience of reading the documentation from the consumer's point of view. As we stated
previously, it is especially helpful when some of the documentation parts are tightly
coupled to the code, as in the form of docstrings. While this approach really makes it easier
to ensure that the source version of the documentation matches with the code it documents,
it does not guarantee that the documentation readership will have access to the latest and
most up-to-date compiled version.

Having only bare source representation is also not enough if the target readers of the
documentation are not proficient enough with command-line tools and will not know how
to build it into a browsable and readable form. This is why it is important to build your
documentation into a consumer-friendly form automatically whenever any change to the
code repository is committed/pushed.

The best way to host the documentation built with Sphinx is to generate an HTML build
and serve it as a static resource with your web server of choice. Sphinx provides a proper
``Makefile`` to build HTML files with the ``make html`` command. Because make is a
very common utility, it should be very easy to integrate this process with any
continuous integration system.

If you are documenting an open source project with Sphinx, then you will make your life a
lot easier by using **Read the Docs** (`<https://readthedocs.org/>`_ ). It is a free service for
hosting the documentation of open source Python projects with Sphinx. The configuration
is completely hassle-free, and it integrates very easily with two popular code hosting
services: GitHub and Bitbucket. In practice, if you have your accounts properly connected
and code repository properly set up, enabling documentation hosting on Read the Docs is a
matter of just a few clicks.
