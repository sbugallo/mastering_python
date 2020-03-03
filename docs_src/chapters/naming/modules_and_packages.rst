6. Modules and packages
***********************

The module and package names inform about the purpose of their content. The names are
short, in lowercase, and usually without underscores, for example:

- ``sqlite``
- ``postgres``
- ``sha1``

They are often suffixed with ``lib`` if they are implementing a protocol, as in the following:

.. code-block:: python

    import smtplib
    import urllib
    import telnetlib

When choosing a name for a module, always consider its content and limit the amount
of redundancy within the whole namespace, for example:

.. code-block:: python

    from widgets.stringwidgets import TextWidget # bad
    from widgets.strings import TextWidget # better

When a module is getting complex and contains a lot of classes, it is a good practice to
create a package and split the module's elements into other modules.

The ``__init__`` module can also be used to put back some common APIs at the top level of
the package. This approach allows you to organize the code into smaller components
without reducing the ease of use.
