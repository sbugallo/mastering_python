1. Interpreters
***************

The reference Python interpreter implementation is called CPython and, as its name
suggests, it is written entirely in the C language. It was always C and probably will be still
for a very long time. That's the implementation that most Python programmers choose
because it is always up to date with the language specification and is the interpreter that
most libraries are tested on. But, besides C, Python interpreter was written in a few other
languages. Also, there are some modified versions of CPython interpreter available under
different names and tailored exactly for some niche applications. Most of them are a few
milestones behind CPython, but provide a great opportunity to use and promote the
language in a specific environment.

1.1. Why should I care?
+++++++++++++++++++++++

There are plenty of alternative Python implementations available. The Python wiki page on
that topic (`https://wiki.python.org/moin/PythonImplementations <https://wiki.python.org/moin/PythonImplementations>`_)
features dozens of different language variants, dialects, or implementations of Python interpreter built with
something other than C. Some of them implement only a subset of the core language
syntax, features, and built-in extensions, but there are at least a few that are almost fully
compatible with CPython. The most important thing to know is that, while some of them
are just toy projects or experiments, most of them were created to solve some real problems:
problems that were either impossible to solve with CPython or required too much of the
developer's effort.

Examples of such problems are as follows:

- Running Python code on embedded systems
- Integration with code written for runtime frameworks, such as Java or .NET, or in different languages
- Running Python code in web browsers

The following sections provide a short description of, subjectively, the most popular and
up-to-date choices that are currently available for Python programmers.

1.2. Stackless Python
+++++++++++++++++++++

Stackless Python advertises itself as an enhanced version of Python. Stackless is named so
because it avoids depending on the C call stack for its own stack. It is, in fact, a modified
CPython code that also adds some new features that were missing from the core Python
implementation at the time Stackless was created. The most important of these are
microthreads, which are managed by the interpreter as cheap and lightweight alternatives
to ordinary threads, that must depend on system kernel context switching and task
scheduling.

The latest available versions are 2.7.15 and 3.6.6 and implement 2.7 and 3.6 versions of
Python, respectively. All the additional features provided by Stackless are exposed as a
framework within this distribution through the built-in stackless module.

Stackless isn't the most popular alternative implementation of Python, but it is worth
knowing, because some of the ideas that were introduced in it had a strong impact on the
language community. The core switching functionality was extracted from Stackless and
published as an independent package named greenlet, which is now the basis for many
useful libraries and frameworks. Also, most of its features were re-implemented in
PyPy: another Python implementation that will be featured later. The official online
documentation of Stackless Python can be found at `https://stackless.readthedocs.io <https://stackless.readthedocs.io>`_
and the project wiki can be found at
`https://github.com/stackless-dev/stackless <https://github.com/stackless-dev/stackless>`_.

1.3. Jython
+++++++++++

Jython is a Java implementation of the language. It compiles the code into Java byte code,
and allows the developers to seamlessly use Java classes within their Python modules.
Jython allows people to use Python as the top-level scripting language on complex
application systems, for example, J2EE. It also brings Java applications into the Python
world. Making Apache Jackrabbit (which is a document repository API based on JCR; see
`http://jackrabbit.apache.org <http://jackrabbit.apache.org>`_  available in a Python program is a good example of what
Jython allows.

The main differences of Jython compared to the CPython implementation are as follows:

- True Java's garbage collection instead of reference counting
- Lack of global interpreter lock (GIL) allows better utilization of multiple cores in multi-threaded applications

The main weakness of this implementation of the language is the lack of support for C
Python Extension APIs, so no Python extensions written in C will work with Jython.

The latest available version of Jython is Jython 2.7, and this corresponds to the 2.7 version
of the language. It is advertised as implementing nearly all of the core Python standard
library and using the same regression test suite. Unfortunately, Jython 3.x was never
released, and the project can be now safely considered dead. However, Jython is still worth
mentioning, even if it is not developed anymore, because it was very unique
implementation at the time and had meaningful impact on other alternative Python
implementations.

The official project page can be found at `http://www.jython.org <http://www.jython.org>`_.

1.4. IronPython
+++++++++++++++

IronPython brings Python into the .NET Framework. The project is supported by Microsoft,
where IronPython's lead developers work. It is quite an important implementation for the
promotion of a language. Besides Java, the .NET community is one of the biggest developer
communities out there. It is also worth noting that Microsoft provides a set of free
development tools that turn Visual Studio into a full-fledged Python IDE. This is
distributed as Visual Studio plugins named Python Tools for Visual Studio (PVTS), and is
available as open source code on GitHub (`http://microsoft.github.io/PTVS <http://microsoft.github.io/PTV>`_).

The latest stable release is 2.7.8, and it is compatible with Python 2.7. Unlike Jython, we can
observe active development on both 2.x and 3.x branches of the interpreter, although
Python 3 support still hasn't been officially released yet. Despite the fact that .NET runs
primarily on Microsoft Windows, it is also possible to run IronPython on macOS and
Linux. This is thanks to Mono, a cross platform, open source .NET implementation.

The main differences and advantages of IronPython compared to CPython are as follows:

- Similar to Jython, the lack of global interpreter lock (GIL) allows for better utilization of multiple cores in multi-threaded applications
- Code written in C# and other .NET languages can be easily integrated in IronPython and vice versa
- It can be run in all major web browsers through Silverlight (although Microsoft will stop supporting Silverlight in 2021)

When speaking about weaknesses, IronPython seems very similar to Jython, because it does
not support the Python/C Extension APIs. This is important for developers who would like
to use packages such as NumPy, which are largely based on C extensions. There were a few
community attempts to bring the Python/C Extensions API support to IronPython, or at
least to provide compatibility for the NumPy package, but unfortunately no project had
notable success in that area.

You can learn more about IronPython from its official project page
at `http://ironpython.net/ <http://ironpython.net/>`_.

1.5. PyPy
+++++++++

PyPy is probably the most exciting alternative implementation of Python, as its goal is to
rewrite Python in Python. PyPy in the Python interpreter is written in Python. We have a C
code layer carrying out the nuts-and-bolts work for CPython. But in PyPy, this C code layer
is rewritten in pure Python.

This means that you can change the interpreter's behavior during execution time, and
implement code patterns that couldn't be easily done in CPython.

PyPy is currently fully compatible with Python version 2.7.13, while the latest PyPy3 is
compatible with Python version 3.5.3.

In the past, PyPy was mostly interesting for theoretical reasons, and it interested those who
enjoyed going deep into the details of the language. It was not generally used in
production, but this has changed through the years. Nowadays, many benchmarks show
that, surprisingly, PyPy is often way faster than the CPython implementation. This project
has its own benchmarking site that tracks performance of each version measured using
dozens of different benchmarks (refer to `http://speed.pypy.org/ <http://speed.pypy.org/>`_). It clearly shows that
PyPy with JIT enabled is usually at least few times faster than CPython. This and other
features of PyPy makes more and more developers decide to use PyPy in their production
environments.

The main differences of PyPy compared to CPython implementation are as follows:

- Garbage collection used instead of reference counting
- Integrated tracing JIT compiler that allows impressive improvements in performance
- Application-level Stackless features borrowed from Stackless Python

Like almost every other alternative Python implementation, PyPy lacks the full official
support of C's Python Extension API. Still, it at least provides some sort of support for C
extensions through its CPyExt subsystem, although it is poorly documented and still not
feature complete. Also, there is an ongoing effort within the community in porting NumPy
to PyPy because it is the most requested feature.

The official PyPy project page can be found at `http://pypy.org <http://pypy.org>`_.

1.6. MicroPython
++++++++++++++++

MicroPython is one of the youngest alternative implementations on that list, as its first
official version was released on May 3, 2014. It is also one of the most interesting
implementations. MicroPython is a Python interpreter that was optimized for use on
microcontrollers and in very constrained environments. Its small size and multiple
optimizations allow it to run in just 256 kilobytes of code space and with just 16 kilobytes of
RAM.

The main reference devices that you can test this interpreter on are BBC's micro:bit devices
and pyboards, which are simple-to-use microcontroller development boards, that are
targeted at teaching programming and electronics.

MicroPython is written in C99 (it's C language standard) and can be built for many
hardware architectures, including x86, x86-64, ARM, ARM Thumb, and Xtensa. It is based
on Python 3, but due to many syntax differences, it's impossible to say that it is fully
compatible with any Python 3.x release. It is certainly a dialect of Python 3, with ``print()``
functions, ``async``/``await`` keywords, and many other Python 3 features, but you can't expect
that your favorite Python 3 libraries will work properly under that interpreter out of the
box.

You can learn more about MicroPython from its official project page at
`https://micropython.org <https://micropython.org>`_.