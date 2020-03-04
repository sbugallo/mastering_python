1. Design principles and unit testing
*************************************

In this section, are first going to take a look at unit testing from a conceptual point of view.
We will revisit some of the software engineering principles we discussed in the previous to
get an idea of how this is related to clean code.

After that, we will discuss in more detail how to put these concepts into practice (at the
code level), and what frameworks and tools we can make use of.

First we quickly define what unit testing is about. Unit tests are parts of the code in charge
of validating other parts of the code. Normally, anyone would be tempted to say that unit
tests, validate the "core" of the application, but such definition regards unit tests to a
secondary place, which is not the way they are thought of. Unit tests are core,
and a critical component of the software and they should be treated with the same
considerations as the business logic.

A unit tests is a piece of code that imports parts of the code with the business logic, and
exercises its logic, asserting several scenarios with the idea to guarantee certain conditions.
There are some traits that unit tests must have, such as:

- Isolation: unit test should be completely independent from any other external agent, and they have to focus only on the business logic. For this reason, they do not connect to a database, they don't perform HTTP requests, etc. Isolation also means that the tests are independent among themselves: they must be able to run in any order, without depending on any previous state.
- Performance: unit tests must run quickly. They are intended to be run multiple times, repeatedly.
- Self-validating: The execution of a unit tests determines its result. There should be no extra step required to interpret the unit test (much less manual).

More concretely, in Python this means that we will have new files where we are going
to place our unit tests, and they are going to be called by some tool. Inside this files we program the tests
themselves. Afterwards, a tool will collect our unit tests and run them, giving a result.

This last part is what self-validation actually means. When the tool calls our files, a Python
process will be launched, and our tests will be running on it. If the tests fail, the process will
have exited with an error code (in a Unix environment, this can be any number different
than 0). The standard is that the tool runs the test, and prints a dot (``.``) for every successful
test, an ``F`` if the test failed (the condition of the test was not satisfied), and an ``E`` if there was
an exception.

1.1. A note about other forms of automated testing
++++++++++++++++++++++++++++++++++++++++++++++++++

Unit tests are intended to verify very small units, for example a function, or a method. We
want from our unit tests to reach a very detailed level of granularity, testing as much code
as possible. To test a class we would not want to use a unit tests, but rather a test suite,
which is a collection of unit tests. Each one of them will be testing something more specific,
like a method of that class.

This is not the only form of unit tests, and it cannot catch every possible error. There are
also acceptance and integration tests, both out of the scope.

In an integration test, we will want to test multiple components at once. In this case we
want to validate if collectively, they work as expected. In this case is acceptable (more than
that, desirable) to have side-effects, and to forget about isolation, meaning that we will
want to issue HTTP requests, connect to databases, and so on.

An acceptance test is an automated form of testing that tries to validate the system from the
perspective of an user, typically executing use cases.

These two last forms of testing lose another nice trait with respect of unit tests: velocity. As
you can imagine, they will take more time to run, therefore they will be run less frequently.

In a good development environment, the programmer will have the entire test suite, and
will run unit tests all the time, repeatedly, while he or she is making changes to the code,
iterating, refactoring, and so on. Once the changes are ready, and the pull request is open, the
continuous integration service will run the build for that branch, where the unit tests will
run, as long as the integration or acceptance tests that might exist. Needless to say, the
status of the build should be successful (green) before merging, but the important part is
the difference between the kind of tests: we want to run unit tests all the time, and less
frequently those test that take longer. For this reason, we want to have a lot of small unit
tests, and a few automated tests, strategically designed to cover as much as possible of
where the unit tests could not reach (the database, for instance).

Finally, a word to the wise. Remember we encourage pragmatism. Besides these
definitions give, and the points made about unit tests in the beginning of the section, the
reader has to keep in mind that the best solution according to your criteria and context,
should predominate. Nobody knows your system better than you. Which means, if for
some reason you have to write an unit tests that needs to launch a Docker container to test
against a database, go for it. Practicality beats purity.

1.2. Unit testing and agile software development
++++++++++++++++++++++++++++++++++++++++++++++++

In modern software development, we want to deliver value constantly, and as quickly as
possible. The rationale behind these goals is that the earlier we get feedback, the less the
impact, and the easier it will be to change. These are no new ideas at all; some of them
resemble manufacturing principles from decades ago, and others (such as the idea of
getting feedback from stakeholders as soon as possible and iterating upon it) you can find
in essays such as The Cathedral and the Bazaar (abbreviated as CatB).

Therefore, we want to be able to respond effectively to changes, and for that, the software
we write will have to change. Like we mentioned in the previous chapters, we want our
software to be adaptable, flexible, and extensible.

The code alone (regardless of how well written and designed it is) cannot guarantee us that
it's flexible enough to be changed. Let's say we design a piece of software following the
SOLID principles, and in one part we actually have a set of components that comply with
the open/closed principle, meaning that we can easily extend them without affecting too
much existing code. Assume further that the code is written in a way that favors
refactoring, so we could change it as required. What's to say that when we make these
changes, we aren't introducing any bugs? How do we know that existing functionality is
preserved? Would you feel confident enough releasing that to your users? Will they believe
that the new version works just as expected?

The answer to all of these questions is that we can't be sure unless we have a formal proof
of it. And unit tests are just that, formal proof that the program works according to the
specification.

Unit (or automated) tests, therefore, work as a safety net that gives us the confidence to
work on our code. Armed with these tools, we can efficiently work on our code, and
therefore this is what ultimately determines the velocity (or capacity) of the team working
on the software product. The better the tests, the more likely it is we can deliver value
quickly without being stopped by bugs every now and then.

1.3. Unit testing and software design
+++++++++++++++++++++++++++++++++++++

This is the other face of the coin when it comes to the relationship between the main code
and unit testing. Besides the pragmatic reasons explored in the previous section, it comes
down to the fact that good software is testable software. **Testability** (the quality attribute
that determines how easy to test software is) is not just a nice to have, but a driver for clean
code.

Unit tests aren't just something complementary to the main code base, but rather something
that has a direct impact and real influence on how the code is written. There are many
levels of this, from the very beginning when we realize that the moment we want to add
unit tests for some parts of our code, we have to change it (resulting in a better version of
it), to its ultimate expression (explored near the end of this chapter) when the entire code
(the design) is driven by the way it's going to be tested via **test-driven design**.

Starting off with a simple example, we will show you a small use case in which tests (and
the need to test our code) lead to improvements in the way our code ends up being written.

In the following example, we will simulate a process that requires sending metrics to an
external system about the results obtained at each particular task (as always, details won't
make any difference as long as we focus on the code). We have a ``Process`` object that
represents some task on the domain problem, and it uses a ``metrics`` client (an external
dependency and therefore something we don't control) to send the actual metrics to the
external entity (that this could be sending data to ``syslog``, or ``statsd``, for instance):

.. code-block:: python

    class MetricsClient:
    """3rd-party metrics client"""
        def send(self, metric_name, metric_value):
            if not isinstance(metric_name, str):
                raise TypeError("expected type str for metric_name")

            if not isinstance(metric_value, str):
                raise TypeError("expected type str for metric_value")

            logger.info(f"sending {metric_name} = {metric_value}")

    class Process:
        def __init__(self):
            self.client = MetricsClient() # A 3rd-party metrics client
        def process_iterations(self, n_iterations):
            for i in range(n_iterations):
                result = self.run_process()
                self.client.send(f"iteration.{i}", result)

In the simulated version of the third-party client, we put the requirement that the
parameters provided must be of type string. Therefore, if the result of the ``run_process``
method is not a string, we might expect it to fail, and indeed it does:

.. code-block:: python

    Traceback (most recent call last):
    ...
    raise TypeError("expected type str for metric_value")
    TypeError: expected type str for metric_value

Remember that this validation is out of our hands and we cannot change the code, so we
must provide the method with parameters of the correct type before proceeding. But since
this is a bug we detected, we first want to write a unit test to make sure it will not happen
again. We do this to actually prove that we fixed the issue, and to protect against this bug in
the future, regardless of how many times the code is refactored.

It would be possible to test the code as is by mocking the client of the ``Process`` object (we
will see how to do so in the section about mock objects, when we explore the tools for unit
testing), but doing so runs more code than is needed (notice how the part we want to test is
nested into the code). Moreover, it's good that the method is relatively small, because if it
weren't, the test would have to run even more undesired parts that we might also need to
mock. This is another example of good design (small, cohesive functions or methods), that
relates to testability.

Finally, we decide not to go to much trouble and test just the part that we need to, so
instead of interacting with the client directly on the main method, we delegate to a
wrapper method, and the new class looks like this:

.. code-block:: python

    class WrappedClient:
        def __init__(self):
            self.client = MetricsClient()
        def send(self, metric_name, metric_value):
            return self.client.send(str(metric_name), str(metric_value))

    class Process:
        def __init__(self):
            self.client = WrappedClient()
            ... # rest of the code remains unchanged

In this case, we opted for creating our own version of the client for metrics, that is, a
wrapper around the third-party library one we used to have. To do this, we place a class
that (with the same interface) will make the conversion of the types accordingly.

This way of using composition resembles the adapter design pattern (we'll explore design
patterns in the next chapter, so, for now, it's just an informative message), and since this is a
new object in our domain, it can have its respective unit tests. Having this object will make
things simpler to test, but more importantly, now that we look at it, we realize that this is
probably the way the code should have been written in the first place. Trying to write a unit
test for our code made us realize that we were missing an important abstraction entirely!

Now that we have separated the method as it should be, let's write the actual unit test for it.
The details about the unittest module used in this example will be explored in more
detail in the part of the chapter where we explore testing tools and libraries, but for now
reading the code will give us a first impression on how to test it, and it will make the
previous concepts a little less abstract:

.. code-block:: python

    import unittest
    from unittest.mock import Mock


    class TestWrappedClient(unittest.TestCase):
        def test_send_converts_types(self):
            wrapped_client = WrappedClient()
            wrapped_client.client = Mock()
            wrapped_client.send("value", 1)
            wrapped_client.client.send.assert_called_with("value", "1")

``Mock`` is a type that's available in the ``unittest.mock`` module, which is a quite convenient
object to ask about all sort of things. For example, in this case, we're using it in place of the
third-party library (mocked into the boundaries of the system, as commented on the next
section) to check that it's called as expected (and once again, we're not testing the library
itself, only that it is called correctly). Notice how we run a call like the one our ``Process``
object, but we expect the parameters to be converted to strings.

1.4. Defining the boundaries of what to test
++++++++++++++++++++++++++++++++++++++++++++

Testing requires effort. And if we are not careful when deciding what to test, we will never
end testing, hence wasting a lot of effort without achieving much.

We should scope the testing to the boundaries of our code. If we don't, we would have to
also test the dependencies (external/third-party libraries or modules) or our code, and then
their respective dependencies, and so on and so forth in a never-ending journey. It's not our
responsibility to test dependencies, so we can assume that these projects have tests of their
own. It would be enough just to test that the correct calls to external dependencies are done
with the correct parameters (and that might even be an acceptable use of patching), but we
shouldn't put more effort in than that.

This is another instance where good software design pays off. If we have been careful in
our design, and clearly defined the boundaries of our system (that is, we designed towards
interfaces, instead of concrete implementations that will change, hence inverting the
dependencies over external components to reduce temporal coupling), then it will be much
more easier to mock these interfaces when writing unit tests.

In good unit testing, we want to patch on the boundaries of our system and focus on the
core functionality to be exercised. We don't test external libraries (third-party tools installed
via ``pip``, for instance), but instead, we check that they are called correctly. When we explore
mock objects later on in this chapter, we will review techniques and tools for performing
these types of assertion.
