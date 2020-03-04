Packaging
=========

This chapter focuses on a repeatable process of writing and releasing Python packages. We
will see how to shorten the time needed to set up everything before starting the real work.
We will also learn how to provide a standardized way to write packages and ease the use of
a test-driven development approach. We will finally learn how to facilitate the release
process.

It is organized into the following four parts:

- A **common pattern** for all packages that describes the similarities between all Python packages, and how distutils and setuptools play a central role the packaging process.
- What are **namespace packages** and why they can be useful?
- How to register and upload packages in the **Python Package Index (PyPI)** with emphasis on security and common pitfalls.
- The **standalone executables** as an alternative way to package and distribute Python applications.

.. include:: creating_package.rst
.. include:: namespace.rst
.. include:: upload.rst
.. include:: executables.rst
