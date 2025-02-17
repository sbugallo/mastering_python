3. Coroutines
*************

As we already know, generator objects are iterables. They implement ``__iter__()`` and
``__next__()``. This is provided by Python automatically so that when we create a generator
object function, we get an object that can be iterated or advanced through the ``next()``
function.

Besides this basic functionality, they have more methods so that they can work as
coroutines. Here, we will explore how generators evolved into coroutines to
support the basis of asynchronous programming before we go into more detail in the next
section, where we explore the new features of Python and the syntax that covers
programming asynchronously. The basic methods added to support
coroutines are as follows:

- ``.close()``
- ``.throw(ex_type[, ex_value[, ex_traceback]])``
- ``.send(value)``

3.1. The methods of the generator interface
+++++++++++++++++++++++++++++++++++++++++++

In this section, we will explore what each of the aforementioned methods does, how it
works, and how it is expected to be used. By understanding how to use these methods, we
will be able to make use of simple coroutines.

Later on, we will explore more advanced uses of coroutines, and how to delegate to sub-
generators (coroutines) in order to refactor code, and how to orchestrate different
coroutines.

3.1.1. close()
--------------

When calling this method, the generator will receive the ``GeneratorExit`` exception. If it's
not handled, then the generator will finish without producing any more values, and its
iteration will stop.

This exception can be used to handle a finishing status. In general, if our coroutine does
some sort of resource management, we want to catch this exception and use that control
block to release all resources being held by the coroutine. In general, it is similar to using a
context manager or placing the code in the finally block of an exception control, but
handling this exception specifically makes it more explicit.

In the following example, we have a coroutine that makes use of a database handler object
that holds a connection to a database, and runs queries over it, streaming data by pages of a
fixed length (instead of reading everything that is available at once):

.. code-block:: python

    def stream_db_records(db_handler):
        try:
            while True:
                yield db_handler.read_n_records(10)
        except GeneratorExit:
            db_handler.close()

At each call to the generator, it will return 10 rows obtained from the database handler, but
when we decide to explicitly finish the iteration and call ``close()``, we also want to close the
connection to the database:

.. code-block:: python

    >>> streamer = stream_db_records(DBHandler("testdb"))
    >>> next(streamer)
    [(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), ...]
    >>> next(streamer)
    [(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), ...]
    >>> streamer.close()
    INFO:...:closing connection to database 'testdb'

Use the ``close()`` method on generators to perform finishing-up tasks
when needed.

3.1.2. throw(ex_type[, ex_value[, ex_traceback]])
-------------------------------------------------

This method will throw the exception at the line where the generator is currently
suspended. If the generator handles the exception that was sent, the code in that
particular except clause will be called, otherwise, the exception will propagate to the
caller.

Here, we are modifying the previous example slightly to show the difference when we use
this method for an exception that is handled by the coroutine, and when it's not:

.. code-block:: python

    class CustomException(Exception):
        pass

    def stream_data(db_handler):
        while True:
            try:
                yield db_handler.read_n_records(10)
            except CustomException as e:
                logger.info(f"controlled error {e}, continuing")
            except Exception as e:
                logger.info(f"unhandled error {e}, stopping")
                db_handler.close()
            break

Now, it is a part of the control flow to receive a ``CustomException``, and, in such a case, the
generator will log an informative message (of course, we can adapt this according to our
business logic on each case), and move on to the next ``yield`` statement, which is the line
where the coroutine reads from the database and returns that data.

This particular example handles all exceptions, but if the last block (``except Exception:``)
wasn't there, the result would be that the generator is raised at the line where the generator
is paused (again, the ``yield``), and it will propagate from there to the caller:

.. code-block:: python

    >>> streamer = stream_data(DBHandler("testdb"))
    >>> next(streamer)
    [(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), (4, 'row 4'), ...]
    >>> next(streamer)
    [(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), (4, 'row 4'), ...]
    >>> streamer.throw(CustomException)
    WARNING:controlled error CustomException(), continuing
    [(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), (4, 'row 4'), ...]
    >>> streamer.throw(RuntimeError)
    ERROR:unhandled error RuntimeError(), stopping
    INFO:closing connection to database 'testdb'
    Traceback (most recent call last):
    ...
    StopIteration

When our exception from the domain was received, the generator continued. However,
when it received another exception that was not expected, the default block caught where
we closed the connection to the database and finished the iteration, which resulted in the
generator being stopped. As we can see from the ``StopIteration`` that was raised, this
generator can't be iterated further.

3.1.3. send(value)
------------------

In the previous example, we created a simple generator that reads rows from a database,
and when we wished to finish its iteration, this generator released the resources linked to
the database. This is a good example of using one of the methods that generators provide
(``close``), but there is more we can do.

An obvious of such a generator is that it was reading a fixed number of rows from the
database.

We would like to parametrize that number so that we can change it throughout
different calls. Unfortunately, the ``next()`` function does not provide us with options for
that. But luckily, we have ``send()``:

.. code-block:: python

    def stream_db_records(db_handler):
        retrieved_data = None
        previous_page_size = 10

        try:
            while True:
                page_size = yield retrieved_data
                if page_size is None:
                    page_size = previous_page_size

                previous_page_size = page_size
                retrieved_data = db_handler.read_n_records(page_size)

        except GeneratorExit:
            db_handler.close()

The idea is that we have now made the coroutine able to receive values from the caller by
means of the ``send()`` method. This method is the one that actually distinguishes a
generator from a coroutine because when it's used, it means that the ``yield`` keyword will
appear on the right-hand side of the statement, and its return value will be assigned to
something else.

In coroutines, we generally find the ``yield`` keyword to be used in the following form:
``receive = yield produced``

The ``yield``, in this case, will do two things. It will send ``produced`` back to the caller, which
will pick it up on the next round of iteration (after calling ``next()``, for example), and it will
suspend there. At a later point, the caller will want to send a value back to the coroutine by
using the ``send()`` method. This value will become the result of the ``yield`` statement,
assigned in this case to the variable named ``receive``.

Sending values to the coroutine only works when this one is suspended at a ``yield``
statement, waiting for something to produce. For this to happen, the coroutine will have to
be advanced to that status. The only way to do this is by calling ``next()`` on it. This means
that before sending anything to the coroutine, this has to be advanced at least once via the
``next()`` method. Failure to do so will result in an exception:

.. code-block:: python

    >>> c = coro()
    >>> c.send(1)
    Traceback (most recent call last):
    ...
    TypeError: can't send non-None value to a just-started generator

.. important:: Always remember to advance a coroutine by calling ``next()`` before sending any values to it.

Back to our example. We are changing the way elements are produced or streamed to make
it able to receive the length of the records it expects to read from the database.

The first time we call ``next()``, the generator will advance up to the line containing ``yield``; it
will provide a value to the caller (``None``, as set in the variable), and it will suspend there).

From here, we have two options. If we choose to advance the generator by calling ``next()``,
the default value of 10 will be used, and it will go on with this as usual. This is because
``next()`` is technically the same as ``send(None)``, but this is covered in the if statement that
will handle the value that we previously set.

If, on the other hand, we decide to provide an explicit value via ``send(<value>)``, this one
will become the result of the ``yield`` statement, which will be assigned to the variable
containing the length of the page to use, which, in turn, will be used to read from the
database.

Successive calls will have this logic, but the important point is that now we can
dynamically change the length of the data to read in the middle of the iteration, at any
point.

Now that we understand how the previous code works, most Pythonistas would expect a
simplified version of it (after all, Python is also about brevity and clean and compact code):

.. code-block:: python

    def stream_db_records(db_handler):
        retrieved_data = None
        page_size = 10
        try:
            while True:
                page_size = (yield retrieved_data) or page_size
                retrieved_data = db_handler.read_n_records(page_size)
        except GeneratorExit:
            db_handler.close()

This version is not only more compact, but it also illustrates the idea better. The parenthesis
around the ``yield`` makes it clearer that it's a statement (think of it as if it were a function
call), and that we are using the result of it to compare it against the previous value.

This works as we expect it does, but we always have to remember to advance the coroutine
before sending any data to it. If we forget to call the first ``next()``, we'll get a ``TypeError``.
This call could be ignored for our purposes because it doesn't return anything we'll use.

It would be good if we could use the coroutine directly, right after it is created without
having to remember to call ``next()`` the first time, every time we are going to use it. Some
authors devised an interesting decorator to achieve this. The idea of this
decorator is to advance the coroutine, so the following definition works automatically:

.. code-block:: python

    @prepare_coroutine
    def stream_db_records(db_handler):
        retrieved_data = None
        page_size = 10
        try:
            while True:
                page_size = (yield retrieved_data) or page_size
                retrieved_data = db_handler.read_n_records(page_size)
        except GeneratorExit:
        db_handler.close()

    >>> streamer = stream_db_records(DBHandler("testdb"))
    >>> len(streamer.send(5))
    5

3.2. More advanced coroutines
+++++++++++++++++++++++++++++

So far, we have a better understanding of coroutines, and we are able to create simple ones
to handle small tasks. We can say that these coroutines are, in fact, just more advanced
generators (and that would be right, coroutines are just fancy generators), but, if we
actually want to start supporting more complex scenarios, we usually have to go for a
design that handles many coroutines concurrently, and that requires more features.

When handling many coroutines, we find new problems. As the control flow of our
application becomes more complex, we want to pass values up and down the stack (as well
as exceptions), be able to capture values from sub-coroutines we might call at any level, and
finally schedule multiple coroutines to run toward a common goal.

To make things simpler, generators had to be extended once again. This is addressed by changing the semantic
of generators so that they are able to return values, and introducing the new yield from construction.

3.2.1. Returning values in coroutines
-------------------------------------

As introduced at the beginning, the iteration is a mechanism that calls
``next()`` on an iterable object many times until a ``StopIteration`` exception is raised.

So far, we have been exploring the iterative nature of generators: we produce values one at
a time, and, in general, we only care about each value as it's being produced at every step of
the ``for`` loop. This is a very logical way of thinking about generators, but coroutines have a
different idea; even though they are technically generators, they weren't conceived with the
idea of iteration in mind, but with the goal of suspending the execution of a code until it's
resumed later on.

This is an interesting challenge; when we design a coroutine, we usually care more about
suspending the state rather than iterating (and iterating a coroutine would be an odd case).
The challenge lies in that it is easy to mix them both. This is because of a technical
implementation detail; the support for coroutines in Python was built upon generators.

If we want to use coroutines to process some information and suspend its execution, it
would make sense to think of them as lightweight threads (or green threads, as they are
called in other platforms). In such a case, it would make sense if they could return values,
much like calling any other regular function.

But let's remember that generators are not regular functions, so in a generator, the
construction ``value = generator()`` will do nothing other than create a generator object.
What would be the semantics for making a generator return a value? It will have to be after
the iteration is done.

When a generator returns a value, its iteration is immediately stopped (it can't be iterated
any further). To preserve the semantics, the ``StopIteration`` exception is still raised, and
the value to be returned is stored inside the exception object. It's the responsibility of the
caller to catch it.

In the following example, we are creating a simple generator that produces two values
and then returns a third. Notice how we have to catch the exception in order to get this
value, and how it's stored precisely inside the exception under the attribute named ``value``:

.. code-block:: python

    >>> def generator():
    ...     yield 1
    ...     yield 2
    ...     return 3
    ...
    >>> value = generator()
    >>> next(value)
    1
    >>> next(value)
    2
    >>> try:
    ...     next(value)
    ... except StopIteration as e:
    ...     print(f">>>>>> returned value {e.value}")
    ...
    >>>>>> returned value 3


3.2.2. Delegating into smaller coroutines: the yield from syntax
-----------------------------------------------------------------

The previous feature is interesting in the sense that it opens up a lot of new possibilities
with coroutines (generators), now that they can return values. But this feature, by itself,
would not be so useful without proper syntax support, because catching the returned value
this way is a bit cumbersome.

This is one of the main features of the yield from syntax. Among other things (that we'll
review in detail), it can collect the value returned by a sub-generator. Remember that we
said that returning data in a generator was nice, but that, unfortunately, writing statements
as ``value = generator()`` wouldn't work. Well, writing it as ``value = yield from
generator()`` would.

3.2.2.1. The simplest use of yield from
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In its most basic form, the new ``yield from`` syntax can be used to chain generators from
nested for loops into a single one, which will end up with a single string of all the values in
a continuous stream.

The canonical example is about creating a function similar to ``itertools.chain()`` from
the standard library. This is a very nice function because it allows you to pass any number
of iterables and will return them all together in one stream.

The naive implementation might look like this:

.. code-block:: python

    def chain(*iterables):
        for it in iterables:
            for value in it:
                yield value

It receives a variable number of iterables, traverses through all of them, and since each
value is iterable, it supports a ``for... in..`` construction, so we have another for loop
to get every value inside each particular iterable, which is produced by the caller function.
This might be helpful in multiple cases, such as chaining generators together or trying to
iterate things that it wouldn't normally be possible to compare in one go (such as lists with
tuples, and so on).

However, the ``yield from`` syntax allows us to go further and avoid the nested loop
because it's able to produce the values from a sub-generator directly. In this case, we could
simplify the code like this:

.. code-block:: python

    def chain(*iterables):
        for it in iterables:
            yield from it

Notice that for both implementations, the behavior of the generator is exactly the same:

.. code-block:: python

    >>> list(chain("hello", ["world"], ("tuple", " of ", "values.")))
    ['h', 'e', 'l', 'l', 'o', 'world', 'tuple', ' of ', 'values.']

This means that we can use ``yield from`` over any other iterable, and it will work as if the
top-level generator (the one the ``yield from`` is using) were generating those values itself.

This works with any iterable, and even generator expressions aren't the exception. Now
that we're familiar with its syntax, let's see how we could write a simple generator function
that will produce all the powers of a number (for instance, if provided with
``all_powers(2, 3)``, it will have to produce 2^0, 2^1,... 2^3 ):

.. code-block:: python

    def all_powers(n, pow):
        yield from (n ** i for i in range(pow + 1))

While this simplifies the syntax a bit, saving one line of a for statement isn't a big
advantage, and it wouldn't justify adding such a change to the language.

Indeed, this is actually just a side effect and the real raison d'être of the ``yield from``
construction is what we are going to explore in the following two sections.

3.2.2.2. Capturing the value returned by a sub-generator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the following example, we have a generator that calls another two nested generators,
producing values in a sequence. Each one of these nested generators returns a value, and
we will see how the top-level generator is able to effectively capture the return value since
it's calling the internal generators through ``yield from``:

.. code-block:: python

    def sequence(name, start, end):
        logger.info(f"{name} started at {start}")
        yield from range(start, end)
        logger.info(f"{name} finished at {end}")
        return end

    def main():
        step1 = yield from sequence("first", 0, 5)
        step2 = yield from sequence("second", step1, 10)
        return step1 + step2

This is a possible execution of the code in main while it's being iterated:

.. code-block:: python

    >>> g = main()
    >>> next(g)
    INFO:generators_yieldfrom_2:first started at 0
    0
    >>> next(g)
    1
    >>> next(g)
    2
    >>> next(g)
    3
    >>> next(g)
    4
    >>> next(g)
    INFO:generators_yieldfrom_2:first finished at 5
    INFO:generators_yieldfrom_2:second started at 5
    5
    >>> next(g)
    6
    >>> next(g)
    7
    >>> next(g)
    8
    >>> next(g)
    9
    >>> next(g)
    INFO:generators_yieldfrom_2:second finished at 10
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    StopIteration: 15

The first line of main delegates into the internal generator, and produces the values,
extracting them directly from it. This is nothing new, as we have already seen. Notice,
though, how the ``sequence()`` generator function returns the end value, which is assigned
in the first line to the variable named ``step1``, and how this value is correctly used at the
start of the following instance of that generator.

In the end, this other generator also returns the second ``end`` value, and the main
generator, in turn, returns the sum of them, which is the value we see once the
iteration has stopped.

.. tip::

    We can use ``yield from`` to capture the last value of a coroutine after it has finished its processing.

3.2.2.3. Sending and receiving data to and from a sub-generator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now, we will see the other nice feature of the ``yield from`` syntax, which is probably what
gives it its full power. As we have already introduced when we explored generators acting
as coroutines, we know that we can send values and throw exceptions at them, and, in such
cases, the coroutine will either receive the value for its internal processing, or it will have to
handle the exception accordingly.

If we now have a coroutine that delegates into other ones (such as in the previous example),
we would also like to preserve this logic. Having to do so manually would be quite
complex if we didn't have this handled by ``yield from`` automatically.

In order to illustrate this, let's keep the same top-level generator (main) unmodified with
respect to the previous example (calling other internal generators), but let's modify the
internal generators to make them able to receive values and handle exceptions. The code is
probably not idiomatic, only for the purposes of showing how this mechanism works:

.. code-block:: python

    def sequence(name, start, end):
        value = start
        logger.info("%s started at %i", name, value)

        while value < end:
            try:
                received = yield value
                logger.info("%s received %r", name, received)
                value += 1

            except CustomException as e:
                logger.info("%s is handling %s", name, e)
                received = yield "OK"

        return end

Now, we will call the main coroutine, not only by iterating it, but also by passing values
and throwing exceptions at it to see how they are handled inside sequence :

.. code-block:: python

    >>> g = main()
    >>> next(g)
    INFO: first started at 0
    0
    >>> next(g)
    INFO: first received None
    1
    >>> g.send("value for 1")
    INFO: first received 'value for 1'
    2
    >>> g.throw(CustomException("controlled error"))
    INFO: first is handling controlled error
    'OK'
    ... # advance more times
    INFO:second started at 5
    5
    >>> g.throw(CustomException("exception at second generator"))
    INFO: second is handling exception at second generator
    'OK'

This example is showing us a lot of different things. Notice how we never send values
to ``sequence``, but only to main, and even so, the code that is receiving those values is the
nested generators. Even though we never explicitly send anything to ``sequence``, it's
receiving the data as it's being passed along by yield from .

The main coroutine calls two other coroutines internally, producing their values, and it will
be suspended at a particular point in time in any of those. When it's stopped at the first one,
we can see the logs telling us that it is that instance of the coroutine that received the value
we sent. The same happens when we throw an exception to it. When the first coroutine
finishes, it returns the value that was assigned in the variable named ``step1``, and passed as
input for the second coroutine, which will do the same (it will handle the ``send()``
and ``throw()`` calls, accordingly).

The same happens for the values that each coroutine produces. When we are at any given
step, the return from calling ``send()`` corresponds to the value that the subcoroutine (the
one that main is currently suspended at) has produced. When we throw an exception that is
being handled, the sequence coroutine produces the value OK, which is propagated to the
called (``main``), and which in turn will end up at main's caller.
