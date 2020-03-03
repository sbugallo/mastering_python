5. Class names
**************

The name of a class has to be concise, precise, and descriptive. A common practice is to use
a suffix that informs about its type or nature, for example:

- SQLEngine
- MimeTypes
- StringWidget
- TestCase

For base or abstract classes, a Base or Abstract prefix can be used as follows:

- BaseCookie
- AbstractFormatter

The most important thing is to be consistent with the class attributes. For example, try to
avoid redundancy between the class and its attributes' names as follows:

.. code-block:: python


    >>> SMTP.smtp_send()  # redundant information in the namespace
    >>> SMTP.send()  # more readable and mnemonic



