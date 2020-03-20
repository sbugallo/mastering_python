5. Building a well-organized documentation system
*************************************************

An easier way to guide your documentation readers and your writers is to provide each
one of them with helpers and guidelines, as we have learned in the previous section of this
chapter.

From a writer's point of view, this is done by having a set of reusable templates, together
with a guide that describes how and when to use them in a project. This is called a
**documentation portfolio**.

From a reader's point of view, it is important to be able to browse the documentation with
no pain, and get used to finding the information efficiently. This is done by building a
**document landscape**.

Obviously, we need to start from guiding documentation writers, because without them,
the readers would not have anything to read. Let's see how such a portfolio looks and how
to build a one.

5.1. Building documentation portfolio
+++++++++++++++++++++++++++++++++++++

There are many kinds of documents a software project can have, from low-level documents
that refer directly to the code, to design papers that provide a high-level overview of the
application.

For instance, Scott Ambler defines an extensive list of document types in his book "Agile
Modeling: Effective Practices for eXtreme Programming and the Unified Process, John Wiley &
Sons". He builds a portfolio from early specifications to operations documents. Even the
project management documents are covered, so the whole documenting needs are built
with a standardized set of templates.
Since a complete portfolio is tightly related to the methodologies used to build the software,
this chapter will only focus on a common subset that you can complete with your specific
needs. Building an efficient portfolio takes a long time, as it captures your working habits.
A common set of documents in software projects can be classified into the following three
categories:

- **Design**: This includes all the documents that provide architectural information
  and low-level design information, such as class diagrams or database diagrams
- **Usage**: This includes all the documents on how to use the software; this can be in
  the shape of a cookbook and tutorials, or a module-level help
- **Operations**: This provides guidelines on how to deploy, upgrade, or operate the
  software

5.1.1. Design
-------------

The important point when creating such documents is to make sure the target readership is
perfectly known, and that the content scope is limited. So, a generic template for design
documents can provide a light structure with a little advice for the writer.
Such a structure might include the following:

- Title
- Author
- Tags (keywords)
- Description (abstract)
- Target (who should read this?)
- Content (with diagrams)
- References to other documents

The content should be three or four pages at most when printed, so be sure to limit the
scope. If it gets bigger, it should be split into several documents or summarized.
The template also provides the author's name and a list of tags to manage its evolutions and
ease its classification. This will be covered later in this chapter.
The example design document template written using reStructuredText markup could be
as follows:

.. code-block:: rst

    =========================================
    Design document title
    =========================================

    :Author: Document Author
    :Tags: document tags separated with spaces

    :abstract:

        Write here a small abstract about your design document.

    .. contents ::

    Audience
    ========

    Explain here who is the target readership.

    Content
    =======

    Write your document here. Do not hesitate to split it in several
    sections.

    References
    ==========

    Put here references, and links to other documents.

5.1.2. Usage
------------

The usage documentation describes how a particular part of the software works. This
documentation can describe low-level parts, such as how a function works, but also high-
level parts, such as command-line arguments for calling the program. This is the most
important part of documentation in framework applications, since the target readership is
mainly the developers that are going to reuse the code.

The three main kinds of documents are as follows:

- **Recipe**: This is a short document that explains how to do something. This kind of
  document targets one readership and focuses on one specific topic.
- **Tutorial**: This is a step-by-step document that explains how to use a feature of
  the software. This document can refer to recipes, and each instance is intended
  for one readership.
- **Module helper**: This is a low-level document that explains what a module
  contains. This document can be shown (for instance) by calling the help built
  into over a module.

5.1.2.1. Recipe
~~~~~~~~~~~~~~~

A recipe answers a very specific problem and provides a solution to resolve it. For example,
ActiveState provides a huge repository of Python recipes online, where developers can
describe how to do something in Python (refer to
`<http://code.activestate.com/recipes/langs/python/>`_). Such a set of recipes related to a
single area/project is often called a *cookbook*.

These recipes must be short and are structured, like this:

- Title
- Submitter
- Last updated
- Version
- Category
- Description
- Source (the source code)
- Discussion (the text explaining the code)
- Comments (from the web)

Often, they are one screen long and do not go into great detail. This structure perfectly fits a
software's needs and can be adapted in a generic structure, where the target readership is
added and the category is replaced by tags:

- Title (short sentence)
- Author
- Tags (keywords)
- Who should read this?
- Prerequisites (other documents to read, for example)
- Problem (a short description)
- Solution (the main text, one or two screens)
- References (links to other documents)

The date and version are not useful here, since project documentation should be managed
like source code in the project. This means that the best way to handle the documentation is
to manage it through the version control system. In most cases, this is exactly the same code
repository as the one that's used for the project's code.

A simple reusable template for the recipes could be as follows:

.. code-block:: rst

    ===========
    Recipe name
    ===========

    :Author: Recipe Author
    :Tags: document tags separated with spaces

    :abstract:

        Write here a small abstract about your design document.

    .. contents ::

    Audience
    ========

    Explain here who is the target readership.

    Prerequisites
    =============

    Write the list of prerequisites for implementing this recipe. This can be
    additional documents, software, specific libraries, environment settings or
    just anything that is required beyond the obvious language interpreter.

    Problem
    =======

    Explain the problem that this recipe is trying to solve.

    Solution
    ========

    Give solution to problem explained earlier. This is the core of a recipe.

    References
    ==========

    Put here references, and links to other documents.

5.1.2.2. Tutorial
~~~~~~~~~~~~~~~~~

A tutorial differs from a recipe in its purpose. It is not intended to resolve an isolated
problem, but rather describes how to use a feature of the application, step by step. This can
be longer than a recipe and can concern many parts of the application. For example, Django
provides a list of tutorials on its website. Writing your first Django App, part 1 (refer to
`<https://docs.djangoproject.com/en/1.9/intro/tutorial01/>`_) explains in a few screens
how to build an application with Django.

A structure for such a document will be as follows:

- Title (short sentence)
- Author
- Tags (words)
- Description (abstract)
- Who should read this?
- Prerequisites (other documents to read, for example)
- Tutorial (the main text)
- References (links to other documents)

5.1.2.3. Module helper
~~~~~~~~~~~~~~~~~~~~~~

The last template that can be added in our collection is the module helper template. A
module helper refers to a single module and provides a description of its contents, together
with usage examples.

Some tools can automatically build such documents by extracting the docstrings and
computing module help using ``pydoc``, such as Epydoc (refer to
`<http://epydoc.sourceforge.net>`_). So, it is possible to generate extensive documentation
based on API introspection. This kind of documentation is often provided in Python
frameworks. For instance, Plone provides a server that keeps an up-to-date collection of
module helpers. You can read more about it at `<http://api.plone.org>`_.

The following are the main problems with this approach:

- There is no smart selection performed over the modules that are really
  interesting to the document
- The code can be obfuscated by the documentation

Furthermore, a module documentation provides examples that sometimes refer to several
parts of the module, and are hard to split between the functions' and classes' docstrings.
The module docstring could be used for that purpose by writing text at the top of the
module. But this ends in having a hybrid file composed of a block of text, then a block of
code. This is rather obfuscating when the code represents less than 50% of the total length.
If you are the author, this is perfectly fine. But when people try to read the code (not the
documentation), they will have to skip the docstrings part.

Another approach is to separate the text in its own file. A manual selection can then be
operated to decide which Python module will have its module helper file. The documents
can then be separated from the code base and allowed to live their own life, as we will see
in the next section. This is how Python is documented.

Many developers will disagree on the fact that doc and code separation is better than
docstrings. This approach means that the documentation process is fully integrated in the
development cycle; otherwise, it will quickly become obsolete. The docstrings approach
solves this problem by providing proximity between the code and its usage example, but
doesn't bring it to a higher level: a document that can be used as part of plain
documentation.

The following template for Module Helper is really simple as it contains just a little
metadata before the content is written. The target is not defined since it is the developers
who wish to use the module:

- Title (module name)
- Author
- Tags (words)
- Content

5.1.3. Operations
-----------------

Operation documents are used to describe how the software can be operated. Consider the
following points:

- Installation and deployment documents
- Administration documents
- Frequently Asked Questions (FAQ) documents
- Documents that explain how people can contribute, ask for help, or provide
  feedback

These documents are very specific, but they can probably use the tutorial template we
defined in the earlier section.

5.2. Your very own documentation portfolio
++++++++++++++++++++++++++++++++++++++++++

The templates that we discussed earlier are just a basis that you can use to document your
software. With time, you will eventually develop your own templates and style for making
documentation. But always keep in mind the light but sufficient approach for project
documentation: each document that's added should have a clearly defined target
readership and should fill a real need. Documents that don't add a real value should not be
written.

Each project is unique and has different documentation needs. For example, small terminal
tools with simple usage can definitely live with only a single ``README`` file as its document
landscape. Having such a minimal single-document approach is completely fine if the
target readers are precisely defined and consistently grouped (system administrators, for
instance).

Also, do not take the provided templates too rigorously. Some additional metadata
provided as an example is really useful in either big projects or in strictly formalized teams.
Tags, for instance, are intended to improve textual searches in big documentations, but will
not provide any value in a documentation landscape consisting only of a few documents.
Also, including a document author is not always a good idea. Such an approach may be
especially questionable in open source projects. In such projects, you will want the
community to also contribute to the documentation. In most cases, such documents are
continuously updated whenever there is such a need by whoever makes the contribution.
People tend to treat the document author as the document owner. This may discourage
people to update the documentation if every document has its author always specified.
Usually, the version control software provides clearer and more transparent information
about real document authors than explicitly provided metadata annotations. The situations
where explicit authors are really recommended are various design documents, especially in
projects where the design process is strictly formalized. The best example of this is the
series of PEP documents provided with the Python language enhancement proposals.

5.3. Building a documentation landscape
+++++++++++++++++++++++++++++++++++++++

The document portfolio we built in the previous section provides a structure at the
document level, but does not provide a way to group and organize it to build the
documentation the readers will have. This is what Andreas Rüping calls a document
landscape, referring to the mental map the readers use when they browse the
documentation. He came up with the conclusion that the best way to organize documents is
to build a logical tree.

In other words, the different kinds of documents composing the portfolio need to find a
place to live within a tree of directories. This place must be obvious to the writers when
they create the document and to the readers when they are looking for it.

A great helper in browsing documentation is the index pages at each level that can drive
writers and readers.

Building a document landscape is done in the following two steps:

- Building a tree for the producers (the writers)
- Building a tree for the consumers (the readers), on top of the producers' tree

This distinction between producers and consumers is important since they access the
documents in different places and different formats.

5.3.1. Producer's layout
------------------------

From a producer's point of view, each document is processed exactly like a Python module.
It should be stored in the version control system and work like code. Writers do not care
about the final appearance of their prose and where it is available; they just want to make
sure that they are writing a document that is the single source of truth on the topic covered.
reStructuredText files stored in a folder tree are available in the version control system,
together with the software code, and are a convenient solution to build the documentation
landscape for producers.

By convention, the docs folder is used as a root of documentation tree, as follows:

.. code-block:: bash

    $ cd my-project
    $ find docs
    docs
    docs/source
    docs/source/design
    docs/source/operations
    docs/source/usage
    docs/source/usage/cookbook
    docs/source/usage/modules
    docs/source/usage/tutorial

Notice that the tree is located in a source folder because the docs folder will be used as a
root folder to set up a special tool in the next section.

From there, an ``index.txt`` file can be added at each level (besides the root), explaining
what kind of documents the folder contains, or summarizing what each subfolder contains.
These index files can define a listing of the documents they contain. For instance, the
operations folder can contain a list of operations documents that are available, as follows:

.. code-block:: rst

    ==========
    Operations
    ==========

    This section contains operations documents:

    - How to install and run the project
    - How to install and manage a database for the project

It is important to know that people tend to forget to update such lists of documents and
tables of contents. So, it is better to have them updated automatically.

5.3.2. Consumer's layout
------------------------

From a consumer's point of view, it is important to work out the index files and to present
the whole documentation in a format that is easy to read and looks good. Web pages are the
best pick and are easy to generate from reStructuredText files.