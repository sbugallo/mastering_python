2. Defensive programming
************************

Defensive programming follows a somewhat different approach than DbC; instead of stating all conditions that
must be held in a contract, that if unmet will raise an exception and make the program fail, this is more
about making all parts of the code (objects, functions, or methods) able to protect themselves against invalid
inputs.

Defensive programming is a technique that has several aspects, and it is particularly useful if it is combined
with other design principles (this means that the fact that it follows a different philosophy than DbC does
not mean that it is a case of either one or the other—it could mean that they might complement each other).

The main ideas on the subject of defensive programming are how to handle errors for scenarios that we might
expect to occur, and how to deal with errors that should never occur (when impossible conditions happen). The
former will fall into error handling procedures, while the latter will be the case for assertions, both topics
we will be exploring in the following sections.

2.1. Error handling
+++++++++++++++++++

In our programs, we resort to error handling procedures for situations that we anticipate as prone to cause
errors. This is usually the case for data input.

The idea behind error handling is to gracefully respond to these expected errors in an attempt to either
continue our program execution or decide to fail if the error turns out to be insurmountable.

There are different approaches by which we can handle errors on our programs, but not all of them are always
applicable. Some of these approaches are as follows:

- Value substitution
- Error logging
- Exception handling

2.1.1. Value substitution
-------------------------

In some scenarios, when there is an error and there is a risk of the software producing an incorrect value or
failing entirely, we might be able to replace the result with another, safer value. We call this value
substitution, since we are in fact replacing the actual erroneous result for a value that is to be considered
non-disruptive (it could be a default, a well-known constant, a sentinel value, or simply something that does
not affect the result at all, like returning zero in a case where the result is intended to be applied to a
sum).

Value substitution is not always possible, however. This strategy has to be carefully chosen for cases where
the substituted value is actually a safe option. Making this decision is a trade-off between robustness and
correctness. A software program is robust when it does not fail, even in the presence of an erroneous
scenario. But this is not correct either. This might not be acceptable for some kinds of software. If the
application is critical, or the data being handled is too sensitive, this is not an option, since we cannot
afford to provide users (or other parts of the application) with erroneous results. In these cases, we opt
for correctness, rather than let the program explode when yielding the wrong results.

A slightly different, and safer, version of this decision is to use default values for data that is not
provided. This can be the case for parts of the code that can work with a default behavior, for example,
default values for environment variables that are not set, for missing entries in configuration files, or for
parameters of functions. We can find examples of Python supporting this throughout different methods of its
API, for example, dictionaries have a get method, whose (optional) second parameter allows you to indicate a
default value:

.. code-block:: python

    >>> configuration = {"dbport": 5432}
    >>> configuration.get("dbhost", "localhost")
    'localhost'
    >>> configuration.get("dbport")
    5432

Environment variables have a similar API:

.. code-block:: python

    >>> import os
    >>> os.getenv("DBHOST")
    'localhost'
    >>> os.getenv("DPORT", 5432)
    5432

In both previous examples, if the second parameter is not provided, None will be returned, because it's the
default value those functions are defined with. We can also define default values for the parameters of our
own functions:

.. code-block:: python

    >>> def connect_database(host="localhost", port=5432):
    ...
            logger.info("connecting to database server at %s:%i", host, port)

In general, replacing missing parameters with default values is acceptable, but substituting erroneous data
with legal close values is more dangerous and can mask some errors. Take this criterion into consideration
when deciding on this approach.

2.1.2. Exception handling
-------------------------

In the presence of incorrect or missing input data, sometimes it is possible to correct the situation with
some examples such as the ones mentioned in the previous section. In other cases, however, it is better to
stop the program from continuing to run with the wrong data than to leave it computing under erroneous
assumptions. In those cases, failing and notifying the caller that something is wrong is a good approach, and
this is the case for a precondition that was violated, as we saw in DbC.

Nonetheless, erroneous input data is not the only possible way in which a function can go wrong. After all,
functions are not just about passing data around; they also have side-effects and connect to external
components.

It could be possible that a fault in a function call is due to a problem on one of these external components,
and not in our function itself. If that is the case, our function should communicate this properly. This will
make it easier to debug. The function should clearly and unambiguously notify the rest of the application
about errors that cannot be ignored so that they can be addressed accordingly.

The mechanism for accomplishing this is an exception. It is important to emphasize that this is what
exceptions should be used for—clearly announcing an exceptional situation, not altering the flow of the
program according to business logic.

If the code tries to use exceptions to handle expected scenarios or business logic, the flow of the program
will become harder to read. This will lead to a situation where exceptions are used as a sort of go-to
statement, that (to make things worse) could span multiple levels on the call stack (up to caller functions),
violating the encapsulation of the logic into its correct level of abstraction. The case could get even worse
if these except blocks are mixing business logic with truly exceptional cases that the code is trying to
defend against; in that case, it will be harder to distinguish between the core logic we have to maintain and
errors to be handled.

.. note::
    Do not use exceptions as a go-to mechanism for business logic. Raise exceptions when there is
    actually something wrong with the code that callers need to be aware of.

This last concept is an important one; exceptions are usually about notifying the caller about something that
is amiss. This means that exceptions should be used carefully because they weaken encapsulation. The more
exceptions a function has, the more the caller function will have to anticipate, therefore knowing about the
function it is calling. And if a function raises too many exceptions, this means that is not so context-free,
because every time we want to invoke it, we will have to keep all of its possible side-effects in mind.

This can be used as a heuristic to tell when a function is not cohesive enough and has too many
responsibilities. If it raises too many exceptions, it could be a sign that it has to be broken down into
multiple, smaller ones.

2.1.2.1. Handle exceptions at the right level of abstraction
............................................................

Exceptions are also part of the principal functions that do one thing, and one thing only. The exception the
function is handling (or raising) has to be consistent with the logic encapsulated on it.

In this example, we can see what we mean by mixing different levels of abstractions. Imagine an object that
acts as a transport for some data in our application. It connects to an external component where the data is
going to be sent upon decoding. In the following listing, we will focus on the deliver_event method:

.. code-block:: python

    class DataTransport:
        """An example of an object handling exceptions of different levels."""
        retry_threshold: int = 5
        retry_n_times: int = 3

        def __init__(self, connector):
            self._connector = connector
            self.connection = None

        def deliver_event(self, event):
            try:
                self.connect()
                data = event.decode()
                self.send(data)
            except ConnectionError as e:
                logger.info(f"connection error detected: {e}")
                raise
            except ValueError as e:
                logger.error(f"{event} contains incorrect data: {e}")
                raise

        def connect(self):
            for _ in range(self.retry_n_times):
                try:
                    self.connection = self._connector.connect()
                except ConnectionError as e:
                    logger.info(f"{e}: attempting new connection in {self.retry_threshold}")
                    time.sleep(self.retry_threshold)
                else:
                    return self.connection

            raise ConnectionError(f"Couldn't connect after {self.retry_n_times} times")

        def send(self, data):
                return self.connection.send(data)

For our analysis, let's zoom in and focus on how the deliver_event() method handles exceptions.

What does ``ValueError`` have to do with ``ConnectionError``? Not much. By looking at these two highly
different types of error, we can get an idea of how responsibilities should be divided. The
``ConnectionError`` should be handled inside the connect method. This will allow a clear separation of
behavior. For example, if this method needs to support retries, that would be a way of doing it. Conversely,
``ValueError`` belongs to the decode method of the event. With this new implementation, this method does not
need to catch any exception: the exceptions it was worrying about before are either handled by internal
methods or deliberately left to be raised.

We should separate these fragments into different methods or functions. For connection management, a small
function should be enough. This function will be in charge of trying to establish the connection, catching
exceptions (should they occur), and logging them accordingly:

.. code-block:: python

    def connect_with_retry(connector, retry_n_times, retry_threshold=5):
        """Tries to establish the connection of <connector> retrying
        <retry_n_times>.
        If it can connect, returns the connection object.
        If it's not possible after the retries, raises ConnectionError
        :param connector: An object with a `.connect()` method.
        :param retry_n_times int: The number of times to try to call
        ``connector.connect()``.
        :param retry_threshold int: The time lapse between retry calls.
        """
        for _ in range(retry_n_times):
        try:
            return connector.connect()
        except ConnectionError as e:
            logger.info(f"{e}: attempting new connection in {retry_threshold}")
            time.sleep(retry_threshold)
        exc = ConnectionError(f"Couldn't connect after {retry_n_times} times")
        logger.exception(exc)
        raise exc

Then, we will call this function in our method. As for the ``ValueError`` exception on the event, we could
separate it with a new object and do composition, but for this limited case it would be overkill, so just
moving the logic to a separate method would be enough. With these two considerations in place, the new version
of the method looks much more compact and easier to read:

.. code-block:: python

    class DataTransport:
        """An example of an object that separates the exception handling by
        abstraction levels.
        """
        retry_threshold: int = 5
        retry_n_times: int = 3

        def __init__(self, connector):
            self._connector = connector
            self.connection = None

        def deliver_event(self, event):
            self.connection = connect_with_retry(self._connector, self.retry_n_times, self.retry_threshold)
            self.send(event)

        def send(self, event):
            try:
                return self.connection.send(event.decode())
            except ValueError as e:
                logger.error(f"{event} contains incorrect data: {e}")
                raise

2.1.2.2 Do not expose tracebacks
................................

This is a security consideration. When dealing with exceptions, it might be acceptable to let them propagate
if the error is too important, and maybe even let the program fail if this is the decision for that particular
scenario and correctness was favored over robustness.

When there is an exception that denotes a problem, it's important to log in with as much detail as possible
(including the traceback information, message, and all we can gather) so that the issue can be corrected
efficiently. At the same time, we want to include as much detail as possible for ourselves: we definitely
don't want any of this becoming visible to users.

In Python, tracebacks of exceptions contain very rich and useful debugging information. Unfortunately, this
information is also very useful for attackers or malicious users who want to try and harm the application, not
to mention that the leak would represent an important information disclosure, jeopardizing the intellectual
property of your organization (parts of the code will be exposed).

If you choose to let exceptions propagate, make sure not to disclose any sensitive information. Also, if you
have to notify users about a problem, choose generic messages (such as Something went wrong, or Page not
found). This is a common technique used in web applications that display generic informative messages when an
HTTP error occurs.

2.1.2.3 Avoid empty except blocks
.................................

This was even referred to as the most diabolical Python anti-pattern. While it is good to anticipate and
defend our programs against some errors, being too defensive might lead to even worse problems. In particular,
the only problem with being too defensive is that there is an empty except block that silently passes without
doing anything.

Python is so flexible that it allows us to write code that can be faulty and yet, will not raise an error,
like this:

.. code-block:: python

    try:
        process_data()
    except:
        pass

The problem with this is that it will not fail, ever. Even when it should. It is also non-Pythonic if you
remember from the zen of Python that errors should never pass silently.

If there is a true exception, this block of code will not fail, which might be what we wanted in the first
place. But what if there is a defect? We need to know if there is an error in our logic to be able to correct
it. Writing blocks such as this one will mask problems, making things harder to maintain.

There are two alternatives:

- Catch a more specific exception (not too broad, such as an Exception). In fact, some linting tools and IDEs will warn you in some cases when the code is handling too broad an exception.
- Do some actual error handling on the except block.

The best thing to do would be to apply both items simultaneously.

Handling a more specific exception (for example, AttributeError or KeyError) will make the program more
maintainable because the reader will know what to expect, and can get an idea of the why of it. It will also
leave other exceptions free to be raised, and if that happens, this probably means a bug, only this time it
can be discovered.

Handling the exception itself can mean multiple things. In its simplest form, it could be just about logging
the exception (make sure to use logger.exception or logger.error to provide the full context of what
happened). Other alternatives could be to return a default value (substitution, only that in this case after
detecting an error, not prior to causing it), or raising a different exception.

.. note::
    If you choose to raise a different exception, to include the original exception that caused the problem,
    which leads us to the next point.

2.1.2.4. Include the original exception
.......................................

As part of our error handling logic, we might decide to raise a different one, and maybe even change its
message. If that is the case, it is recommended to include the original exception that led to that.

In Python 3, we can now use the raise <e> from <original_exception> syntax. When using this construction, the
original traceback will be embedded into the new exception, and the original exception will be set in the
``__cause__`` attribute of the resulting one.

For example, if we desire to wrap default exceptions with custom ones internally to our project, we could
still do that while including information about the root exception:

.. code-block:: python

    class InternalDataError(Exception):
        """An exception with the data of our domain problem."""
        def process(data_dictionary, record_id):
            try:
                return data_dictionary[record_id]
            except KeyError as e:
                raise InternalDataError("Record not present") from e

.. note:: Always use the raise <e> from <o> syntax when changing the type of the exception.

2.2. Using assertions in Python
+++++++++++++++++++++++++++++++

Assertions are to be used for situations that should never happen, so the expression on the assert statement
has to mean an impossible condition. Should this condition happen, it means there is a defect in the software.

In contrast with the error handling approach, here there is (or should not be) a possibility of continuing the
program. If such an error occurs, the program must stop. It makes sense to stop the program because, as
commented before, we are in the presence of a defect, so there is no way to move forward by releasing a new
version of the software that corrects this defect.

The idea of using assertions is to prevent the program from causing further damage if such an invalid scenario
is presented. Sometimes, it is better to stop and let the program crash, rather than let it continue
processing under the wrong assumptions.

For this reason, assertions should not be mixed with the business logic, or used as control flow mechanisms
for the software. The following example is a bad idea:

.. code-block:: python

    try:
        assert condition.holds(), "Condition is not satisfied"
    except AssertionError:
        alternative_procedure()


.. note:: Do not catch the AssertionError exception.

Make sure that the program terminates when an assertion fails.

Include a descriptive error message in the assertion statement and log the errors to make sure that you can
properly debug and correct the problem later on.

Another important reason why the previous code is a bad idea is that besides catching AssertionError, the
statement in the assertion is a function call. Function calls can have side-effects, and they aren't always
repeatable (we don't know if calling condition.holds() again will yield the same result). Moreover, if we stop
the debugger at that line, we might not be able to conveniently see the result that causes the error, and,
again, even if we call that function again, we don't know if that was the offending value.

A better alternative requires fewer lines of code and provides more useful information:

.. code-block:: python

    result = condition.holds()
    assert result > 0, "Error with {0}".format(result)
