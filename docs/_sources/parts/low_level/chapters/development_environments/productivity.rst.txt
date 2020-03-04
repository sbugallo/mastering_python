3. Popular productivity tools
*****************************

Productivity tool is bit of a vague term. On one hand, almost every open source code
package that has been released and is available online is a kind of productivity booster: it
provides ready-to-use solutions to some problem, so that no one needs to spend time on it
(ideally speaking). On the other hand, you could say that the whole of Python is about
productivity, and both are undoubtedly true. Almost everything in this language and
community surrounding it seems to be designed in order to make software development as
productive as possible.

This creates a positive feedback loop. Since writing code is fun and easy, a lot of
programmers use their free time to create tools that make it even easier and fun. And this
fact will be used here as a basis for a very subjective and non-scientific definition of a
productivity tool: a piece of software that makes development easier and more fun.

By nature, productivity tools focus mainly on certain elements of the development process,
such as testing, debugging, and managing packages, and are not core parts of products that
they help to build. In some cases, they may not even be referred to anywhere in the project's
codebase, despite being used on a daily basis.

The most important productivity tools, ``pip`` and ``venv``, were already discussed earlier.
Some of them have packages for specific problems, such as profiling and testing,
which have their own chapters in this book. This section is dedicated to other tools that are
really worth mentioning, but have no specific chapter in this book where they could be
introduced.

3.1. Custom Python shells
+++++++++++++++++++++++++

Python programmers spend a lot of time in interactive interpreter sessions. It is very good
for testing small code snippets, accessing documentation, or even debugging code at
runtime. The default interactive Python session is very simple, and does not provide many
features, such as tab completion or code introspection helpers. Fortunately, the default
Python shell can be easily extended and customized.

If you use an interactive shell very often, you can easily modify the behavior of it prompt.
Python at startup reads the ``PYTHONSTARTUP`` environment variable, looking for the path of
the custom initializations script. Some operating system distributions where Python is a
common system component (for example, Linux, macOS) may be already preconfigured to
provide a default startup script. It is commonly found in the users, home directory under
the ``.pythonstartup`` name. These scripts often use the ``readline`` module (based on the
GNU readline library) together with ``rlcompleter`` in order to provide interactive tab
completion and command history.

If you don't have a default python startup script, you can easily build your own. A basic
script for command history and tab completion can be as simple as the following:

.. code-block:: python

    import atexit
    import os


    try:
        import readline
    except ImportError:
        print("Completion unavailable: readline module not available")
    else:
         import rlcompleter
         # tab completion
         readline.parse_and_bind('tab: complete')
         # Path to history file in user's home directory.
         # Can use your own path.
         history_file = os.path.join(os.environ['HOME'], '.python_shell_history')
         try:
            readline.read_history_file(history_file)
         except IOError:
            pass

         atexit.register(readline.write_history_file, history_file)
         del os, history_file, readline, rlcompleter

Create this file in your home directory and call it ``.pythonstartup``. Then, add
a ``PYTHONSTARTUP`` variable in your environment using the path of your file.

3.1.1. Setting up the PYTHONSTARTUP environment variable
--------------------------------------------------------

If you are running Linux or macOS, the simplest way is to create the startup script in your
home folder. Then, link it with a ``PYTHONSTARTUP`` environment variable that's been set in
the system shell startup script. For example, Bash and Korn shell use the ``.profile`` file,
where you can insert a line, as follows:

.. code-block:: bash

    export PYTHONSTARTUP=~/.pythonstartup

If you are running Windows, it is easy to set a new environment variable as an
administrator in the system preferences, and save the script in a common place instead of
using a specific user location.

Writing on the PYTHONSTARTUP script may be a good exercise, but creating a good custom
shell all alone is a challenge that only a few can find time for. Fortunately, there are a few
custom Python shell implementations that immensely improve the experience of interactive
sessions in Python.

3.1.2. IPython
--------------

IPython
(`https://ipython.readthedocs.io/en/stable/overview.html <https://ipython.readthedocs.io/en/stable/overview.html>`_)
provides an extended Python command shell. Among the features that it provides, the most interesting
ones are as follows:

- Dynamic object introspection
- System shell access from the prompt
- Profiling direct support
- Debugging facilities

Now, IPython is a part of the larger project called Jupyter, which provides interactive
notebooks with live code that can be written in many different languages.

3.1.3. bpython
--------------

bpython (`https://bpython-interpreter.org/ <https://bpython-interpreter.org/>`_) advertises itself as a fancy interface
to the Python interpreter. Here are some of the accented features on the projects page:

- In-line syntax highlighting
- Readline-like autocomplete with suggestions displayed as you type
- Expected parameter list for any Python function
- Auto-indentation
- Python 3 support

3.1.4. ptpython
---------------

ptpython (`https://github.com/jonathanslenders/ptpython/ <https://github.com/jonathanslenders/ptpython/>`_) is another
approach to the topic of advanced Python shells. What is interesting about this project is that core prompt
utilities implementation is available as a separate package, called ``prompt_toolkit`` (from
the same author). This allows us to easily create various aesthetically pleasing interactive
command-line interfaces.

It is often compared to bpython in functionalities, but the main difference is that it enables
compatibility mode with IPython and its syntax enables additional features, such
as %pdb, %cpaste, or %profile.

3.2. Incorporating shells in your own scripts and programs
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Sometimes, there is a need to incorporate a read-eval-print loop (REPL), similar to Python's
interactive session, inside of your own software. This allows for easier experimentation
with your code and inspection of its internal state. The simplest module that allows for
emulating Python's interactive interpreter already comes with the standard library and is
named code.

The script that starts interactive sessions consists of one import and single function call:

.. code-block:: python

    import code
    code.interact()

You can easily do some minor tuning, such as modify a prompt value or add banner and
exit messages, but anything more fancy will require a lot more work. If you want to have
more features, such as code highlighting, completion, or direct access to the system shell, it
is always better to use something that was already built by someone. Fortunately, all of the
interactive shells that were mentioned in the previous section can be embedded in your
own program as easily as the code module.

The following are examples of how to invoke all of the previously mentioned shells inside
of your code:

.. code-block:: python

    import IPython
    IPython.embed()

    import bpython
    bpython.embed()

    from ptpython.repl import embed
    embed(globals(), locals())

3.3. Interactive debuggers
++++++++++++++++++++++++++

Code debugging is an integral element of the software development process. Many
programmers can spend most of their life using only extensive logging
and print statements as their primary debugging tools, but most professional developers
prefer to rely on some kind of debugger.

Python already ships with a built-in interactive debugger called ``pdb`` (refer
to `https://docs.python.org/3/library/pdb.html <https://docs.python.org/3/library/pdb.html>`_). It can be invoked from
the command line on the existing script, so Python will enter post-mortem debugging if the program exits
abnormally:

.. code-block:: python

    python -m pdb script.py

Post-mortem debugging, while useful, does not cover every scenario. It is useful only when
the application exits with some exception if the bug occurs. In many cases, faulty code just
behaves abnormally, but does not exit unexpectedly. In such cases, custom breakpoints can
be set on a specific line of code using this single-line idiom:

.. code-block:: python

    import pdb
    pdb.set_trace()

This will cause the Python interpreter to start the debugger session on this line during
runtime.

``pdb`` is very useful for tracing issues, and at first glance it may look very familiar to the wellknown
GNU Debugger (GDB). Because Python is a dynamic language, the ``pdb`` session is
very similar to an ordinary interpreter session. This means that the developer is not limited
to tracing code execution, but can call any code and even perform module imports.

Sadly, because of its roots (``bdb``), your first experience with ``pdb`` can be a bit overwhelming
due to the existence of cryptic short-letter debugger commands such as ``h``, ``b``, ``s``, ``n``, ``j``, and ``r``.
When in doubt, the ``help pdb`` command which can be typed during the debugger session,
will provide extensive usage and additional information.

The debugger session in ``pdb`` is also very simple and does not provide additional features
such as tab completion or code highlighting. Fortunately, there are a few packages available
on PyPI that provide such features from alternative Python shells, as mentioned in the
previous section. The most notable examples are as follows:

- ``ipdb``: This is a separate package based on ipython
- ``ptpdb``: This is a separate package based on ptpython
- ``bpdb``: This is bundled with bpython
