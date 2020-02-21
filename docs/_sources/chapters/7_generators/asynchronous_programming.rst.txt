4. Asynchronous programming
***************************

With the constructions we have seen so far, we are able to create asynchronous programs in
Python. This means that we can create programs that have many coroutines, schedule them
to work in a particular order, and switch between them when they're suspended after a
``yield from`` has been called on each of them.

The main advantage that we can take out of this is the possibility of parallelizing I/O
operations in a non-blocking way. What we would need is a low-level generator (usually
implemented by a third-party library) that knows how to handle the actual I/O while the
coroutine is suspended. The idea is for the coroutine to effect suspension so that our
program can handle another task in the meantime. The way the application would retrieve
the control back is by means of the ``yield from`` statement, which will suspend and
produce a value to the caller (as in the examples we saw previously when we used this
syntax to alter the control flow of the program).

This is roughly the way asynchronous programming had been working in Python for quite
a few years, until it was decided that better syntactic support was needed.

The fact that coroutines and generators are technically the same causes some confusion.
Syntactically (and technically), they are the same, but semantically, they are different. We
create generators when we want to achieve efficient iteration. We typically create
coroutines with the goal of running non-blocking I/O operations.

While this difference is clear, the dynamic nature of Python would still allow developers to
mix these different type of objects, ending up with a runtime error at a very late stage of the
program. Remember that in the simplest and most basic form of the ``yield from`` syntax,
we used this construction over iterables (we created a sort of chain function applied over
strings, lists, and so on). None of these objects were coroutines, and it still worked. Then,
we saw that we can have multiple coroutines, use ``yield from`` to send the value (or
exceptions), and get some results back. These are clearly two very different use cases,
however, if we write something along the lines of the following statement:
``result = yield from iterable_or_awaitable()``

It's not clear what ``iterable_or_awaitable returns``. It can be a simple iterable such as a
string, and it might still be syntactically correct. Or, it might be an actual coroutine. The cost
of this mistake will be paid much later.

For this reason, the typing system in Python had to be extended. Before Python 3.5,
coroutines were just generators with a @coroutine decorator applied, and they were to be
called with the yield from syntax. Now, there is a specific type of object, that is, a
coroutine.

This change heralded, syntax changes as well. The ``await`` and ``async def`` syntax were
introduced. The former is intended to be used instead of ``yield from``, and it only works
with awaitable objects (which coroutines conveniently happen to be). Trying to
call ``await`` with something that doesn't respect the interface of an awaitable will raise an
exception. The ``async def`` is the new way of defining coroutines, replacing the
aforementioned decorator, and this actually creates an object that, when called, will return
an instance of a coroutine.

Without going into all the details and possibilities of asynchronous programming in
Python, we can say that despite the new syntax and the new types, this is not doing
anything fundamentally different from concepts we have covered.

The idea of programming asynchronously in Python is that there is an event loop
(typically ``asyncio`` because it's the one that is included in the standard library, but there
are many others that will work just the same) that manages a series of coroutines. These
coroutines belong to the event loop, which is going to call them according to its scheduling
mechanism. When each one of these runs, it will call our code (according to the logic we
have defined inside the coroutine we programmed), and when we want to get control back
to the event loop, we call ``await <coroutine>``, which will process a task asynchronously.
The event loop will resume and another coroutine will take place while that operation is left
running.

In practice, there are more particularities and edge cases that are beyond the scope. It is, however, worth
mentioning that these concepts are related to the ideas introduced in this chapter and that this arena is
another place where generators demonstrate being a core concept of the language, as there are many things
constructed on top of them.
