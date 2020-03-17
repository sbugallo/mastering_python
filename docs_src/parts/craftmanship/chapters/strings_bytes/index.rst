Strings and bytes
=================

The topic of strings may provide some confusion for programmers that used to program
only in Python 2. In Python 3, there is only one datatype capable of storing textual
information. It is ``str``, or simply string. It is an immutable sequence that stores Unicode
code points. This is the major difference from Python 2, where ``str`` represented byte strings:
something that is now handled by the ``bytes`` objects (but not exactly in the same way).

Strings in Python are sequences. This single fact should be enough to include them in a
section covering other container types. But they differ from other container types in one
important detail. Strings have very specific limitations on what type of data they can store,
and that is Unicode text.

``bytes``, and its mutable alternative, ``bytearray``, differs from ``str`` by allowing only bytes as
a sequence value, and bytes in Python are integers in the ``0 <= x < 256`` range. This may
be a bit confusing at the beginning, because, when printed, they may look very similar to
strings:

.. code-block:: python

    >>> print(bytes([102, 111, 111]))
    b'foo'

The ``bytes`` and ``bytearray`` types allow you to work with raw binary data that may not
always have to be textual (for example, audio/video files, images, and network packets).
The true nature of these types is revealed when they are converted into other sequence
types, such as list or tuple:

.. code-block:: python

    >>> list(b'foo bar')
    [102, 111, 111, 32, 98, 97, 114]
    >>> tuple(b'foo bar')
    (102, 111, 111, 32, 98, 97, 114)

A lot of Python 3 controversy was about breaking the backwards compatibility for string
literals and how Python deals with Unicode. Starting from Python 3.0, every string literal
without any prefix is Unicode. So, literals enclosed by single quotes (``'``), double quotes (``"``),
or groups of three quotes (single or double) without any prefix represent the ``str`` data type:

.. code-block:: python

    >>> type("some string")
    <class 'str'>

In Python 2, the Unicode literals required a ``u`` prefix (like ``u"some string"``). This prefix is
still allowed for backwards compatibility (starting from Python 3.3), but does not hold any
syntactic meaning in Python 3.

Byte literals were already presented in some of the previous examples, but let's explicitly
present their syntax for the sake of consistency. Bytes literals are enclosed by single quotes,
double quotes, or triple quotes, but must be preceded with a ``b`` or ``B`` prefix:

.. code-block:: python

    >>> type(b"some bytes")
    <class 'bytes'>

Note that Python does not provide a syntax for ``bytearray`` literals. If you want to create
a ``bytearray`` value, you need to use a ``byte``s literal and a ``bytearray()`` type constructor:

.. code-block:: python

    >>> bytearray(b'some bytes')
    bytearray(b'some bytes')

It is important to remember that Unicode strings contain abstract text that is independent
from the byte representation. This makes them unable to be saved on the disk or sent over
the network without encoding them to binary data. There are two ways to encode string
objects into byte sequences:

- Using the ``str.encode(encoding, errors)`` method, which encodes the string using a registered codec for encoding. Codec is specified using the encoding argument, and, by default, it is 'utf-8'. The second argument, errors, specifies the error handling scheme. It can be 'strict' (default), 'ignore' , 'replace' , 'xmlcharrefreplace', or any other registered handler (refer to the built-in codecs module documentation).
- Using the ``bytes(source, encoding, errors``) constructor, which creates a new bytes sequence. When the source is of the ``str`` type, then the encoding argument is obligatory and it does not have a default value. The usage of the encoding and errors arguments is the same as for the ``str.encode()`` method.

Binary data represented by ``bytes`` can be converted into a string in an analogous way:

- Using the ``bytes.decode(encoding, errors)`` method, which decodes the bytes using the codec registered for encoding. The arguments of this method have the same meaning and defaults as the arguments of ``str.encode()``.
- Using the ``str(source, encoding, error)`` constructor, which creates a new string instance. Similar to the ``bytes()`` constructor, the encoding argument in the ``str()`` call has no default value and must be provided if the bytes sequence is used as a source.

.. tip::

    Due to changes made in Python 3, some people tend to refer to
    the ``bytes`` instances as byte strings. This is mostly due to historic reasons:
    ``bytes`` in Python 3 is the sequence type that is the closest one to
    the ``str`` type from Python 2 (but not the same). Still, the ``bytes`` instance is
    a sequence of bytes and also does not need to represent textual data. So, in
    order to avoid any confusion, it is advised to always refer to them as
    either bytes or byte sequence, despite their similarities to strings. The
    concept of strings is reserved for textual data in Python 3, and this is now
    always ``str``.

1. Implementation details
*************************

Python strings are immutable. This is also true for byte sequences. This is an important fact,
because it has both advantages and disadvantages. It also affects the way strings should be
handled in Python efficiently. Thanks to immutability, strings can be used as dictionary
keys or set collection elements because, once initialized, they will never change their
value. On the other hand, whenever a modified string is required (even with only tiny
modification), a completely new instance needs to be created. Fortunately, ``bytearray``, as a
mutable version of ``bytes``, does not have such an issue. Byte arrays can be modified inplace
(without creating new objects) through item assignments and can be dynamically
resized, exactly like lists: using appends, pops, inserts, and so on.

2. String concatenation
***********************

The fact that Python strings are immutable imposes some problems when multiple string
instances need to be joined together. As we stated previously, concatenating immutable
sequences results in the creation of a new sequence object. Consider that a new string is
built by repeated concatenation of multiple strings, as follows:

.. code-block:: python

    substrings = ["These ", "are ", "strings ", "to ", "concatenate."]
    s = ""

    for substring in substrings:
        s += substring

This will result in quadratic runtime costs in the total string length. In other words, it is
highly inefficient. For handling such situations, the ``str.join()`` method is available. It
accepts iterables of strings as the argument and returns joined strings. The call to ``join()`` of
the str type can be done in two forms:

.. code-block:: python

    # using empty literal
    s = "".join(substrings)

    # using "unbound" method call
    str.join("", substrings)

The first form of the ``join()`` call is the most common idiom. The string that provides this
method will be used as a separator between concatenated substrings. Consider the
following example:

.. code-block:: python

    >>> ','.join(['some', 'comma', 'separated', 'values'])
    'some,comma,separated,values'

It is worth remembering that just because it is faster (especially for large lists), it does not
mean that the ``join()`` method should be used in every situation where two strings need to
be concatenated. Despite being a widely recognized idiom, it does not improve code
readability. And readability counts! There are also some situations where ``join()`` may not
perform as well as ordinary concatenation with a + operator. Here are some examples:

- If the number of substrings is very small and they are not contained already by some iterable variable (existing list or tuple of strings): in some cases the overhead of creating a new sequence just to perform concatenation can overshadow the gain of using ``join()``.
- When concatenating short literals: thanks to some interpreter-level optimizations, such as constant folding in CPython, some complex literals (not only strings), such as ``'a' + 'b' +'c'``, can be translated into a shorter form at compile time (here ``'abc'``). Of course, this is enabled only for constants (literals) that are relatively short.

Ultimately, if the number of strings to concatenate is known beforehand, the best
readability is ensured by proper string formatting either using the ``str.format()`` method,
the ``%`` operator, or f-string formatting. In code sections where the performance is not critical
or the gain from optimizing string concatenation is very little, string formatting is
recommended as the best alternative to concatenation.

2.1. Constant folding, the peephole optimizer, and the AST optimizer
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

CPython uses various techniques to optimize your code. The first optimization takes place
as soon as source code is transformed into the form of the abstract syntax tree, just before it
is compiled into byte code. CPython can recognize specific patterns in the abstract syntax
tree and make direct modifications to it. The other kind of optimizations are handled by the
peephole optimizer. It implements a number of common optimizations directly on Python's
byte code. As we mentioned earlier, constant folding is one such feature. It allows the
interpreter to convert complex literal expressions (such as ``"one" + " " + "thing"``,
``" " * 79``, or ``60 * 1000``) into a single literal that does not require any additional operations
(concatenation or multiplication) at runtime.

Until Python 3.5, all constant folding was done in CPython only by the peephole optimizer.
For strings, the resulting constants were limited in length by a hardcoded value. In Python
3.5, this value was equal to 20. In Python 3.7, most of the constant folding optimizations are
handled earlier on the abstract syntax tree level. These particular details are a curiosity
rather than a thing that can be relied on in your day-to-day programming. Information
about other interesting optimizations performed by AST and peephole optimizers can be
found in the *Python/ast_opt.c* and *Python/peephole.c* files of Python's source code.

3. String formatting with f-strings
***********************************

F-strings are one of the most beloved new Python features that came with Python 3.6. It's
also one of the most controversial features of that release. The f-strings or formatted string
literals that were introduced by the PEP 498 document add a new way to format strings in
Python. Before Python 3.6, there were two basic ways to format strings:

- Using ``%`` formatting for example ``"Some string with included % value" % "other"``
- Using the ``str.format()`` method for example ``"Some string with included {other} value".format(other="other")``

Formatted string literals are denoted with the ``f`` prefix, and their syntax is closest to the
``str.format()`` method, as they use a similar markup for denoting replacement fields in
the text that has to be formatted. In the ``str.format()`` method, the text substitutions refer
to arguments and keyword arguments that are passed to the formatting method. You can
use either anonymous substitutions that will translate to consecutive argument indexes,
explicit argument indexes, or keyword names.

This means that the same string can be formatted in different ways:

.. code-block:: python

    >>> from sys import version_info
    >>> "This is Python {}.{}".format(*version_info)
    'This is Python 3.7'
    >>> "This is Python {0}.{1}".format(*version_info)
    'This is Python 3.7'
    >>> "This is Python {major}.{minor}".format(major=version_info.major, minor=version_info.minor)
    'This is Python 3.7'

What makes f-strings special is that replacement fields can be any Python expression, and it
will be evaluated at runtime. Inside of strings, you have access to any variable that is
available in the same namespace as the formatted literal. With f-strings, the preceding
examples could be written in the following way:

.. code-block:: python

    >>> from sys import version_info
    >>> f"This is Python {version_info.major}.{version_info.minor}"
    'This is Python 3.7'

The ability to use expressions as replacement fields make formatting code simpler and
shorter. You can also use the same formatting specifiers of replacement fields (for padding,
aligning, signs, and so on) as the ``str.format()`` method, and the syntax is as follows:

.. code-block:: python

    f"{replacement_field_expression:format_specifier}"

The following is a simple example of code that prints the first ten powers of the number 10
using f-strings and aligns results using string formatting with padding:

.. code-block:: python

    >>> for x in range(10):
    ...     print(f"10^{x} == {10**x:10d}")
    ...
    10^0 == 1
    10^1 == 10
    10^2 == 100
    10^3 == 1000
    10^4 == 10000
    10^5 == 100000
    10^6 == 1000000
    10^7 == 10000000
    10^8 == 100000000
    10^9 == 1000000000

The full formatting specification of the Python string is almost like a separate minilanguage
inside Python. The best reference for it is the official documentation which you
can find under `https://docs.python.org/3/library/string.html <https://docs.python.org/3/library/string.html>`_.
Another useful internet resource for that topic is `https://pyformat.info/ <https://pyformat.info/>`_, which presents
the most important elements of this specification using practical examples.
