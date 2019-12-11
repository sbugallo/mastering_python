2. Context managers
*******************

Context managers are quite useful since they correctly respond to a pattern. The pattern is actually every situation
where we want to run some code, and has preconditions and postconditions, meaning that we want to run things before and
after a certain main action.

Most of the time, we see context managers around resource management. For example, on situations when we open files, we
want to make sure that they are closed after processing (so we do not leak file descriptors), or if we open a connection
to a service (or even a socket), we also want to be sure to close it accordingly, or when removing temporary files, and
so on.

In all of these cases, you would normally have to remember to free all of the resources that were allocated and that is
just thinking about the best case—but what about exceptions and error handling? Given the fact that handling all
possible combinations and execution paths of our program makes it harder to debug, the most common way of addressing
this issue is to put the cleanup code on a ``finally`` block so that we are sure we do not miss it. For example, a very
simple case would look like the following:

.. code-block:: python

    fd = open(filename)
    try:
        process_file(fd)
    finally:
        fd.close()

Nonetheless, there is a much elegant and Pythonic way of achieving the same thing:

.. code-block:: python

    with open(filename) as fd:
        process_file(fd)

The with statement enters the context manager. In this case, the open function implements the context manager protocol,
which means that the file will be automatically closed when the block is finished, even if an exception occurred.

Context managers consist of two magic methods: ``__enter__`` and ``__exit__``. On the first line of the context manager,
the with statement will call the first method, ``__enter__``, and whatever this method returns will be assigned to the
variable labeled after as. This is optional—we don't really need to return anything specific on the ``__enter__``
method, and even if we do, there is still no strict reason to assign it to a variable if it is not required.

After this line is executed, the code enters a new context, where any other Python code can be run. After the last
statement on that block is finished, the context will be exited, meaning that Python will call the ``__exit__`` method
of the original context manager object we first invoked.

If there is an exception or error inside the context manager block, the ``__exit__`` method will still be called, which
makes it convenient for safely managing cleaning up conditions. In fact, this method receives the exception that was
triggered on the block in case we want to handle it in a custom fashion.

Despite the fact that context managers are very often found when dealing with resources,
this is not the sole application they have. We can implement our own context managers in order to handle the particular
logic we need.

Context managers are a good way of separating concerns and isolating parts of the code that should be kept independent,
because if we mix them, then the logic will become harder to maintain.

As an example, consider a situation where we want to run a backup of our database with a script. The caveat is that the
backup is offline, which means that we can only do it while the database is not running, and for this we have to stop
it. After running the backup, we want to make sure that we start the process again, regardless of how the process of the
backup itself went. Now, the first approach would be to create a huge monolithic function that tries to do everything
in the same place, stop the service, perform the backup task, handle exceptions and all possible edge cases, and then
try to restart the service again. You can imagine such a function, and for that reason, I will spare you the details,
and instead come up directly with a possible way of tackling this issue with context managers:

.. code-block:: python

    class DBHandler:

        def stop_database():
            run("systemctl stop postgresql.service")

        def start_database():
            run("systemctl start postgresql.service")

        def __enter__(self):
            self.stop_database()
            return self

        def __exit__(self, exc_type, ex_value, ex_traceback):
            self.start_database()

    def db_backup():
        run("pg_dump database")

    def main():
        with DBHandler():
            db_backup()

As a general rule, it should be good practice (although not mandatory), to always return something on the ``__enter__``.

Notice the signature of the ``__exit__`` method. It receives the values for the exception that was raised on the block.
If there was no exception on the block, they are all none.

The return value of ``__exit__`` is something to consider. Normally, we would want to leave the method as it is, without
returning anything in particular. If this method returns True, it means that the exception that was potentially raised
will not propagate to the caller and will stop there. Sometimes, this is the desired effect, maybe even depending on the
type of exception that was raised, but in general it is not a good idea to swallow the exception. Remember: errors
should never pass silently.

Keep in mind not to accidentally return True on the ``__exit__``. If you do, make sure that this is exactly what you
want, and that there is a good reason for it.

2.1. Implementing context managers
++++++++++++++++++++++++++++++++++

In general, we can implement context managers implementing the ``__enter__`` and ``__exit__`` magic methods, and then
that object will be able to support the context manager protocol. While this is the most common way for context managers
to be implemented, it is not the only one.

The ``contextlib`` module contains a lot of helper functions and objects to either implement context managers or use
some already provided ones that can help us write more compact code.

Let's start by looking at the ``contextmanager`` decorator. When the ``contextlib.contextmanager`` decorator is applied
to a function, it converts the code on that function into a context manager. The function in question has to be a
particular kind of function called a generator function, which will separate the statements into what is going to be on
the ``__enter__`` and ``__exit__`` magic methods, respectively.

The equivalent code of the previous example can be rewritten with the ``contextmanager`` decorator like this:

.. code-block:: python

    import contextlib

    @contextlib.contextmanager
    def db_handler():
        stop_database()
        yield
        start_database()

    with db_handler():
     db_backup()

Here, we define the generator function and apply the ``@contextlib.contextmanager`` decorator to it. The function
contains a yield statement, which makes it a generator function. Again, details on generators are not relevant in this case. All we need to know is
that when this decorator is applied, everything before the yield statement will be run as if it were part of the
``__enter__`` method. Then, the yielded value is going to be the result of the context manager evaluation (what
``__enter__`` would return), and what would be assigned to the variable if we chose to assign it.

At that point, the generator function is suspended, and the context manager is entered, where, again, we run the backup
code for our database. After this completes, the execution resumes, so we can consider that every line that comes after
the yield statement will be part of the ``__exit__`` logic.

Another helper we could use is ``contextlib.ContextDecorator``. This is a mixin base class that provides the logic for
applying a decorator to a function that will make it run inside the context manager, while the logic for the context
manager itself has to be provided by implementing the aforementioned magic methods.

In order to use it, we have to extend this class and implement the logic on the required methods:

.. code-block:: python

    class dbhandler_decorator(contextlib.ContextDecorator):

        def __enter__(self):
            stop_database()

        def __exit__(self, ext_type, ex_value, ex_traceback):
            start_database()

    @dbhandler_decorator()
    def offline_backup():
        run("pg_dump database")

There is no with statement. We just have to call the function, ``and offline_backup()`` will automatically run inside a
context manager. This is the logic that the base class provides to use it as a decorator that wraps the original
function so that it runs inside a context manager.

The only downside of this approach is that by the way the objects work, they are completely independent (the decorator
doesn't know anything about the function that is decorating, and vice versa. This, however good, means that you cannot
get an object that you would like to use inside the context manager, so if you really need to use the object returned by
the ``__exit__`` method, one of the previous approaches will have to be the one of choice.

Being a decorator also poses the advantage that the logic is defined only once, and we can reuse it as many times as we
want by simply applying the decorators to other functions that require the same invariant logic.


Note that ``contextlib.suppress`` is a util package that enters a context manager, which, if one of the provided
exceptions is raised, doesn't fail. It's similar to running that same code on a try/except block and passing an
exception or logging it, but the difference is that calling the suppress method makes it more explicit that those
exceptions that are controlled as part of our logic. For example, consider the following code:

.. code-block:: python

    import contextlib

    with contextlib.suppress(DataConversionException):
         parse_data(input_json_or_dict)

Here, the presence of the exception means that the input data is already in the expected format, so there is no need for
conversion, hence making it safe to ignore it.