6. Code generation
******************
As we already mentioned, the dynamic code generation is the most difficult approach to
metaprogramming. There are tools in Python that allow you to generate and execute code
or even do some modifications to the already compiled code objects.

Various projects such as **Hy** show that even whole languages can be
reimplemented in Python using code generation techniques. This proves that the
possibilities are practically limitless. Knowing how vast this topic is and how badly it is
riddled with various pitfalls, I won't even try to give detailed suggestions on how to create
code this way, or to provide useful code samples.

Anyway, knowing what is possible may be useful for you if you plan to study this field
deeper by yourself. So, treat this section only as a short summary of possible starting points
for further learning.

6.1. exec, eval and compile
+++++++++++++++++++++++++++

Python provides the following three built-in functions to manually execute, evaluate, and
compile arbitrary Python code:

- ``exec(object, globals, locals)``: This allows you to dynamically execute the Python code. ``object`` should be a string or code object (see the ``compile()`` function) representing a single statement or sequence of multiple statements. The ``globals`` and ``locals`` arguments provide global and local namespaces for the executed code and are optional. If they are not provided, then the code is executed in the current scope. If provided, ``globals`` must be a dictionary, while ``locals`` might be any mapping object; it always returns ``None``.
- ``eval(expression, globals, locals)``: This is used to evaluate the given expression by returning its value. It is similar to ``exec()``, but it expects ``expression`` to be a single Python expression and not a sequence of statements. It returns the value of the evaluated expression.
- ``compile(source, filename, mode)``: This compiles the source into the code object or AST object. The source code is provided as a string value in the ``source`` argument. The filename should be the file from which the code was read. If it has no file associated (for example, because it was created dynamically), then ``<string>`` is the value that is commonly used. Mode should be either ``exec`` (sequence of statements), ``eval`` (single expression), or ``single`` (a single interactive statement, such as in a Python interactive session).

The ``exec()`` and ``eval()`` functions are the easiest to start with when trying to dynamically
generate code because they can operate on strings. If you already know how to program in
Python, then you may already know how to correctly generate working source code
programmatically.

The most useful in the context of metaprogramming is obviously ``exec()`` because it allows
you to execute any sequence of Python statements. The word *any* should be alarming for
you. Even ``eval()``, which allows only evaluation of expressions in the hands of a skillful
programmer (when fed with the user input), can lead to serious security holes. Note that
crashing the Python interpreter is the scenario you should be least afraid of. Introducing
vulnerability to remote execution exploits due to irresponsible use
of ``exec()`` and ``eval()`` can cost you your image as a professional developer, or even your
job.

Even if used with a trusted input, there is a list of little details
about ``exec()`` and ``eval()`` that is too long to be included here, but might affect how your
application works in ways you would not expect. Armin Ronacher has a good article that
lists the most important of them, titled "Be careful with exec and eval" in Python (refer
to `http://lucumr.pocoo.org/2011/2/1/exec-in-python/ <http://lucumr.pocoo.org/2011/2/1/exec-in-python/>`_ ).

Despite all these frightening warnings, there are natural situations where the usage
of ``exec()`` and ``eval()`` is really justified. Still, in the case of even the tiniest doubt, you
should not use them and try to find a different solution.

.. tip::

    The signature of the ``eval()`` function might make you think that if you
    provide empty ``globals`` and ``locals`` namespaces and wrap it with
    proper ``try`` ... ``except`` statements, then it will be reasonably safe. There
    could be nothing more wrong. Ned Batcheler has written a very good
    article in which he shows how to cause an interpreter segmentation fault
    in the ``eval()`` call, even with erased access to all Python built-ins
    (see `http://nedbatchelder.com/blog/201206/eval_really_is_dangerous.html <http://nedbatchelder.com/blog/201206/eval_really_is_dangerous.html>`_ ).
    This is single proof that both ``exec()`` and ``eval()`` should never
    be used with untrusted input.

6.2. Abstract syntax tree (AST)
+++++++++++++++++++++++++++++++

The Python syntax is converted into AST before it is compiled into byte code. This is a tree
representation of the abstract syntactic structure of the source code. Processing of Python
grammar is available thanks to the built-in ``ast`` module. Raw ASTs of Python code can be
created using the ``compile()`` function with the ``ast.PyCF_ONLY_AST`` flag, or by using
the ``ast.parse()`` helper. Direct translation in reverse is not that simple and there is no
function provided in the standard library that can do so. Some projects, such as PyPy, do
such things though.

The ``ast`` module provides some helper functions that allow you to work with the AST, for
example:

.. code-block:: python

    >>> tree = ast.parse('def hello_world(): print("hello world!")')
    >>> tree
    <_ast.Module object at 0x00000000038E9588>
    >>> ast.dump(tree)
    "Module(
        body=[
            FunctionDef(
                name='hello_world',
                args=arguments(
                    args=[],
                    vararg=None,
                    kwonlyargs=[],
                    kw_defaults=[],
                    kwarg=None,
                    defaults=[]
                ),
                body=[
                    Expr(
                        value=Call(
                            func=Name(id='print', ctx=Load()),
                            args=[Str(s='hello world!')],
                            keywords=[]
                        )
                    )
                ],
                decorator_list=[],
                returns=None
            )
        ]
    )"

The output of ``ast.dump()`` in the preceding example was reformatted to increase the
readability and better show the tree-like structure of the AST. It is important to know that
the AST can be modified before being passed to ``compile()``. This gives you many new
possibilities. For instance, new syntax nodes can be used for additional instrumentation,
such as test coverage measurement. It is also possible to modify the existing code tree in
order to add new semantics to the existing syntax. Such a technique is used by the MacroPy
project (`https://github.com/lihaoyi/macropy <https://github.com/lihaoyi/macropy>`_)
to add syntactic macros to Python using the already existing syntax:

.. figure:: ../../_static/images/macropy_diagram.jpg
    :width: 50%
    :align: center

AST can also be created in a purely artificial manner, and there is no need to parse any
source at all. This gives Python programmers the ability to create Python bytecode for
custom domain-specific languages, or even completely implement other programming
languages on top of Python VMs.

6.2.1. Import hooks
-------------------

Taking advantage of MacroPy's ability to modify original ASTs would be as easy as using
the ``import macropy.activate`` statement if it could somehow override the Python
import behavior. Fortunately, Python provides a way to intercept imports using the
following two kinds of import hooks:

- **Meta hooks**: These are called before any other ``import`` processing has occurred. Using meta hooks, you can override the way in which ``sys.path`` is processed for even frozen and built-in modules. To add a new meta hook, a new **meta path finder** object must be added to the ``sys.meta_path`` list.
- **Import path hooks**: These are called as part of ``sys.path`` processing. They are used if the path item associated with the given hook is encountered. The import path hooks are added by extending the ``sys.path_hooks`` list with a new **path finder** object.

The details of implementing both path finders and meta path finders are extensively
implemented in the official Python documentation
(see `https://docs.python.org/3/reference/import.html <https://docs.python.org/3/reference/import.html>`_ ).
The official documentation should be your primary resource if you want to interact with imports on that level.
This is so because import machinery in Python is rather complex and any attempt to summarize it
in a few paragraphs would inevitably fail. Here, we just noted that such things are possible.

6.3. Projects that use code generation patterns
+++++++++++++++++++++++++++++++++++++++++++++++

It is hard to find a really usable implementation of the library that relies on code generation
patterns that is not only an experiment or simple proof of concept. The reasons for that
situation are fairly obvious:

- Deserved fear of the ``exec()`` and ``eval()`` functions because, if used irresponsibly, they can cause real disasters
- Successful code generation is very difficult to develop and maintain because it requires a deep understanding of the language and exceptional programming skills in general

Despite these difficulties, there are some projects that successfully take this approach either
to improve performance or achieve things that would be impossible by other means.

6.3.1. Falcon's compiled router
-------------------------------

Falcon ( `http://falconframework.org/ <http://falconframework.org/>`_ ) is a minimalist Python WSGI web
framework for building fast and lightweight APIs. It strongly encourages the REST architectural style that
is currently very popular around the web. It is a good alternative to other rather heavy
frameworks, such as Django or Pyramid. It is also a strong competitor to other micro-
frameworks that aim for simplicity, such as Flask, Bottle, or web2py.

One of its features is it's very simple routing mechanism. It is not as complex as the routing
provided by Django ``urlconf`` and does not provide as many features, but in most cases is
just enough for any API that follows the REST architectural design. What is most
interesting about Falcon's routing is the internal construction of that router. Falcon's router
is implemented using the code generated from the list of routes, and code changes every
time a new route is registered. This is the effort that's needed to make routing fast.

Consider this very short API example, taken from Falcon's web documentation:

.. code-block:: python


    import falcon
    import json


    class QuoteResource:
        def on_get(self, req, resp):
            """Handles GET requests"""

            quote = {
                'quote': 'I\'ve always been more interested in '
                         'the future than in the past.',
                'author': 'Grace Hopper'
            }

            resp.body = json.dumps(quote)

        api = falcon.API()
        api.add_route('/quote', QuoteResource())

In short, the highlighted call to the ``api.add_route()`` method updates dynamically the
whole generated code tree for Falcon's request router. It also compiles it using
the ``compile()`` function and generates the new route-finding function using ``eval()``. Let's
take a closer look at the following ``__code__`` attribute of
the ``api._router._find()`` function:

.. code-block:: python

    >>> api._router._find.__code__
    <code object find at 0x00000000033C29C0, file "<string>", line 1>
    >>> api.add_route('/none', None)
    >>> api._router._find.__code__
    <code object find at 0x00000000033C2810, file "<string>", line 1>

This transcript shows that the code of this function was generated from the string and not
from the real source code file (the "``<string>``" file). It also shows that the actual code
object changes with every call to the ``api.add_route()`` method (the object's address in
memory changes).

6.3.2. Hy
---------

Hy (`http://docs.hylang.org/ <http://docs.hylang.org/>`_) is the dialect of Lisp, and is written entirely in
Python. Many similar projects that implement other code in Python usually try only to tokenize the
plain form of code that's provided either as a file-like object or string and interpret it as a
series of explicit Python calls. Unlike others, Hy can be considered as a language that runs
fully in the Python runtime environment, just like Python does. Code written in Hy can use
the existing built-in modules and external packages and vice-versa. Code written with Hy
can be imported back into Python.

To embed Lisp in Python, Hy translates Lisp code directly into Python AST. Import
interoperability is achieved using the import hook that is registered once the Hy module is
imported into Python. Every module with the ``.hy`` extension is treated as the Hy module
and can be imported like the ordinary Python module. The following is a *hello world*
program written in this Lisp dialect:

.. code-block:: lisp

    ;; hyllo.hy
    (defn hello [] (print "hello world"))

It can be imported and executed with the following Python code:

.. code-block:: python

    >>> import hy
    >>> import hyllo
    >>> hyllo.hello()
    hello world

If we dig deeper and try to disassemble ``hyllo.hello`` using the built-in ``dis`` module, we
will notice that the byte code of the Hy function does not differ significantly from its pure
Python counterpart, as shown in the following code:

.. code-block:: python

    >>> import dis
    >>> dis.dis(hyllo.hello)
    2           0 LOAD_GLOBAL   0 (print)
                3 LOAD_CONST    1 ('hello world!')
                6 CALL_FUNCTION 1 (1 positional, 0 keyword pair)
                9 RETURN_VALUE
    >>> def hello(): print("hello world!")
    ...
    >>> dis.dis(hello)
    1           0 LOAD_GLOBAL   0 (print)
                3 LOAD_CONST    1 ('hello world!')
                6 CALL_FUNCTION 1 (1 positional, 0 keyword pair)
                9 POP_TOP       10 LOAD_CONST
                0 (None)        13 RETURN_VALUE
