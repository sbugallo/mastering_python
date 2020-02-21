2. Frameworks and tools for testing
***********************************

There are a lot of tools we can use for writing out unit tests, all of them with pros and cons
and serving different purposes. But among all of them, there are two that will most likely
cover almost every scenario, and therefore we limit this section to just them.
Along with testing frameworks and test running libraries, it's often common to find projects
that configure code coverage, which they use as a quality metric. Since coverage (when
used as a metric) is misleading, after seeing how to create unit tests we'll discuss why it's
not to be taken lightly.

Frameworks and libraries for unit testing
In this section, we will discuss two frameworks for writing and running unit tests. The first
one, unittest, is available in the standard library of Python, while the second
one, pytest, has to be installed externally via pip .
unittest : https:/​ / ​ docs.​ python.​ org/​ 3/​ library/​ unittest.​ html
pytest : https:/​ / ​ docs.​ pytest.​ org/​ en/​ latest/
When it comes to covering testing scenarios for our code, unittest alone will most likely
suffice, since it has plenty of helpers. However, for more complex systems on which we
have multiple dependencies, connections to external systems, and probably the need to
patch objects, and define fixtures parameterize test cases, then pytest looks like a more
complete option.

We will use a small program as an example to show you how could it be tested using both
options which in the end will help us to get a better picture of how the two of them
compare.
The example demonstrating testing tools is a simplified version of a version control tool
that supports code reviews in merge requests. We will start with the following criteria:
A merge request is rejected if at least one person disagrees with the changes
If nobody has disagreed, and the merge request is good for at least two other
developers, it's approved
In any other case, its status is pending
And here is what the code might look like:
from enum import Enum
class MergeRequestStatus(Enum):
APPROVED = "approved"
REJECTED = "rejected"
PENDING = "pending"
class MergeRequest:
def __init__(self):
self._context = {
"upvotes": set(),
"downvotes": set(),
}
@property
def status(self):
if self._context["downvotes"]:
return MergeRequestStatus.REJECTED
elif len(self._context["upvotes"]) >= 2:
return MergeRequestStatus.APPROVED
return MergeRequestStatus.PENDING
def upvote(self, by_user):
self._context["downvotes"].discard(by_user)
self._context["upvotes"].add(by_user)
def downvote(self, by_user):
self._context["upvotes"].discard(by_user)
self._context["downvotes"].add(by_user)
[ 227 ]Unit Testing and Refactoring
Chapter 8
unittest
The unittest module is a great option with which to start writing unit tests because it
provides a rich API to write all kinds of testing conditions, and since it's available in the
standard library, it's quite versatile and convenient.
The unittest module is based on the concepts of JUnit (from Java), which in turn is also
based on the original ideas of unit testing that come from Smalltalk, so it's object-oriented in
nature. For this reason, tests are written through objects, where the checks are verified by
methods, and it's common to group tests by scenarios in classes.
To start writing unit tests, we have to create a test class that inherits from
unittest.TestCase, and define the conditions we want to stress on its methods. These
methods should start with test_*, and can internally use any of the methods inherited
from unittest.TestCase to check conditions that must hold true.
Some examples of conditions we might want to verify for our case are as follows:
class TestMergeRequestStatus(unittest.TestCase):
def test_simple_rejected(self):
merge_request = MergeRequest()
merge_request.downvote("maintainer")
self.assertEqual(merge_request.status, MergeRequestStatus.REJECTED)
def test_just_created_is_pending(self):
self.assertEqual(MergeRequest().status, MergeRequestStatus.PENDING)
def test_pending_awaiting_review(self):
merge_request = MergeRequest()
merge_request.upvote("core-dev")
self.assertEqual(merge_request.status, MergeRequestStatus.PENDING)
def test_approved(self):
merge_request = MergeRequest()
merge_request.upvote("dev1")
merge_request.upvote("dev2")
self.assertEqual(merge_request.status, MergeRequestStatus.APPROVED)
The API for unit testing provides many useful methods for comparison, the most common
one being assertEquals(<actual>, <expected>[, message]), which can be used to
compare the result of the operation against the value we were expecting, optionally using a
message that will be shown in the case of an error.
[ 228 ]Unit Testing and Refactoring
Chapter 8
Another useful testing method allows us to check whether a certain exception was raised or
not. When something exceptional happens, we raise an exception in our code to prevent
continuous processing under the wrong assumptions, and also to inform the caller that
something is wrong with the call as it was performed. This is the part of the logic that ought
to be tested, and that's what this method is for.
Imagine that we are now extending our logic a little bit further to allow users to close their
merge requests, and once this happens, we don't want any more votes to take place (it
wouldn't make sense to evaluate a merge request once this was already closed). To prevent
this from happening, we extend our code and we raise an exception on the unfortunate
event when someone tries to cast a vote on a closed merge request.
After adding two new statuses ( OPEN and CLOSED ), and a new close() method, we
modify the previous methods for the voting to handle this check first:
class MergeRequest:
def __init__(self):
self._context = {
"upvotes": set(),
"downvotes": set(),
}
self._status = MergeRequestStatus.OPEN
def close(self):
self._status = MergeRequestStatus.CLOSED
...
def _cannot_vote_if_closed(self):
if self._status == MergeRequestStatus.CLOSED:
raise MergeRequestException("can't vote on a closed merge
request")
def upvote(self, by_user):
self._cannot_vote_if_closed()
self._context["downvotes"].discard(by_user)
self._context["upvotes"].add(by_user)
def downvote(self, by_user):
self._cannot_vote_if_closed()
self._context["upvotes"].discard(by_user)
self._context["downvotes"].add(by_user)
[ 229 ]Unit Testing and Refactoring
Chapter 8
Now, we want to check that this validation indeed works. For this, we're going to use
the asssertRaises and assertRaisesRegex methods:
def test_cannot_upvote_on_closed_merge_request(self):
self.merge_request.close()
self.assertRaises(
MergeRequestException, self.merge_request.upvote, "dev1"
)
def test_cannot_downvote_on_closed_merge_request(self):
self.merge_request.close()
self.assertRaisesRegex(
MergeRequestException,
"can't vote on a closed merge request",
self.merge_request.downvote,
"dev1",
)
The former will expect that the provided exception is raised when calling the callable in the
second argument, with the arguments ( *args and **kwargs ) on the rest of the function,
and if that's not the case it will fail, saying that the exception that was expected to be raised,
wasn't. The latter does the same but it also checks that the exception that was raised,
contains the message matching the regular expression that was provided as a parameter.
Even if the exception is raised, but with a different message (not matching the regular
expression), the test will fail.
Try to check for the error message, as not only will the exception, as an
extra check, be more accurate and ensure that it is actually the exception
we want that is being triggered, it will check whether another one of the
same types got there by chance.
Parametrized tests
Now, we would like to test how the threshold acceptance for the merge request works, just
by providing data samples of what the context looks like without needing the entire
MergeRequest object. We want to test the part of the status property that is after the line
that checks if it's closed, but independently.
The best way to achieve this is to separate that component into another class, use
composition, and then move on to test this new abstraction with its own test suite:
class AcceptanceThreshold:
def __init__(self, merge_request_context: dict) -> None:
self._context = merge_request_context
[ 230 ]Unit Testing and Refactoring
Chapter 8
def status(self):
if self._context["downvotes"]:
return MergeRequestStatus.REJECTED
elif len(self._context["upvotes"]) >= 2:
return MergeRequestStatus.APPROVED
return MergeRequestStatus.PENDING
class MergeRequest:
...
@property
def status(self):
if self._status == MergeRequestStatus.CLOSED:
return self._status
return AcceptanceThreshold(self._context).status()
With these changes, we can run the tests again and verify that they pass, meaning that this
small refactor didn't break anything of the current functionality (unit tests ensure
regression). With this, we can proceed with our goal to write tests that are specific to the
new class:
class TestAcceptanceThreshold(unittest.TestCase):
def setUp(self):
self.fixture_data = (
(
{"downvotes": set(), "upvotes": set()},
MergeRequestStatus.PENDING
),
(
{"downvotes": set(), "upvotes": {"dev1"}},
MergeRequestStatus.PENDING,
),
(
{"downvotes": "dev1", "upvotes": set()},
MergeRequestStatus.REJECTED
),
(
{"downvotes": set(), "upvotes": {"dev1", "dev2"}},
MergeRequestStatus.APPROVED
),
)
def test_status_resolution(self):
for context, expected in self.fixture_data:
with self.subTest(context=context):
status = AcceptanceThreshold(context).status()
self.assertEqual(status, expected)
[ 231 ]Unit Testing and Refactoring
Chapter 8
Here, in the setUp() method, we define the data fixture to be used throughout the tests. In
this case, it's not actually needed, because we could have put it directly on the method, but
if we expect to run some code before any test is executed, this is the place to write it,
because this method is called once before every test is run.
By writing this new version of the code, the parameters under the code being tested are
clearer and more compact, and at each case, it will report the results.
To simulate that we're running all of the parameters, the test iterates over all the data, and
exercises the code with each instance. One interesting helper here is the use of subTest,
which in this case we use to mark the test condition being called. If one of these iterations
failed, unittest would report it with the corresponding value of the variables that were
passed to the subTest (in this case, it was named context, but any series of keyword
arguments would work just the same). For example, one error occurrence might look like
this:
FAIL: (context={'downvotes': set(), 'upvotes': {'dev1', 'dev2'}})
----------------------------------------------------------------------
Traceback (most recent call last):
File "" test_status_resolution
self.assertEqual(status, expected)
AssertionError: <MergeRequestStatus.APPROVED: 'approved'> !=
<MergeRequestStatus.REJECTED: 'rejected'>
If you choose to parameterize tests, try to provide the context of each
instance of the parameters with as much information as possible to make
debugging easier.
pytest
Pytest is a great testing framework, and can be installed via pip install pytest . A
difference with respect to unittest is that, while it's still possible to classify test scenarios
in classes and create object-oriented models of our tests, this is not actually mandatory, and
it's possible to write unit tests with less boilerplate by just checking the conditions we want
to verify with the assert statement.
By default, making comparisons with an assert statement will be enough for pytest to
identify a unit test and report its result accordingly. More advanced uses such as those seen
in the previous section are also possible, but they require using specific functions from the
package.
[ 232 ]Unit Testing and Refactoring
Chapter 8
A nice feature is that the command pytests will run all the tests that it can discover, even
if they were written with unittest . This compatibility makes it easier to transition
from unittest toward pytest gradually.
Basic test cases with pytest
The conditions we tested in the previous section can be rewritten in simple functions with
pytest .
Some examples with simple assertions are as follows:
def test_simple_rejected():
merge_request = MergeRequest()
merge_request.downvote("maintainer")
assert merge_request.status == MergeRequestStatus.REJECTED
def test_just_created_is_pending():
assert MergeRequest().status == MergeRequestStatus.PENDING
def test_pending_awaiting_review():
merge_request = MergeRequest()
merge_request.upvote("core-dev")
assert merge_request.status == MergeRequestStatus.PENDING
Boolean equality comparisons don't require more than a simple assert statement, whereas
other kinds of checks like the ones for the exceptions do require that we use some functions:
def test_invalid_types():
merge_request = MergeRequest()
pytest.raises(TypeError, merge_request.upvote, {"invalid-object"})
def test_cannot_vote_on_closed_merge_request():
merge_request = MergeRequest()
merge_request.close()
pytest.raises(MergeRequestException, merge_request.upvote, "dev1")
with pytest.raises(
MergeRequestException,
match="can't vote on a closed merge request",
):
merge_request.downvote("dev1")
[ 233 ]Unit Testing and Refactoring
Chapter 8
In this case, pytest.raises is the equivalent of unittest.TestCase.assertRaises,
and it also accepts that it be called both as a method and as a context manager. If we want
to check the message of the exception, instead of a different method
(like assertRaisesRegex ), the same function has to be used, but as a context manager,
and by providing the match parameter with the expression we would like to identify.
pytest will also wrap the original exception into a custom one that can be expected (by
checking some of its attributes such as .value, for instance) in case we want to check for
more conditions, but this use of the function covers the vast majority of cases.
Parametrized tests
Running parametrized tests with pytest is better, not only because it provides a cleaner
API, but also because each combination of the test with its parameters generates a new test
case.
To work with this, we have to use the pytest.mark.parametrize decorator on our test.
The first parameter of the decorator is a string indicating the names of the parameters to
pass to the test function, and the second has to be iterable with the respective values for
those parameters.
Notice how the body of the testing function is reduced to one line (after removing the
internal for loop, and its nested context manager), and the data for each test case is
correctly isolated from the body of the function, making it easier to extend and maintain:
@pytest.mark.parametrize("context,expected_status", (
(
{"downvotes": set(), "upvotes": set()},
MergeRequestStatus.PENDING
),
(
{"downvotes": set(), "upvotes": {"dev1"}},
MergeRequestStatus.PENDING,
),
(
{"downvotes": "dev1", "upvotes": set()},
MergeRequestStatus.REJECTED
),
(
{"downvotes": set(), "upvotes": {"dev1", "dev2"}},
MergeRequestStatus.APPROVED
),
))
def test_acceptance_threshold_status_resolution(context, expected_status):
assert AcceptanceThreshold(context).status() == expected_status
[ 234 ]Unit Testing and Refactoring
Chapter 8
Use @pytest.mark.parametrize to eliminate repetition, keep the body
of the test as cohesive as possible, and make the parameters (test inputs or
scenarios) that the code must support explicitly.
Fixtures
One of the great things about pytest is how it facilitates creating reusable features so that
we can feed our tests with data or objects in order to test more effectively and without
repetition.
For example, we might want to create a MergeRequest object in a particular state, and use
that object in multiple tests. We define our object as a fixture by creating a function and
applying the @pytest.fixture decorator. The tests that want to use that fixture will have
to have a parameter with the same name as the function that's defined, and pytest will
make sure that it's provided:
@pytest.fixture
def rejected_mr():
merge_request = MergeRequest()
merge_request.downvote("dev1")
merge_request.upvote("dev2")
merge_request.upvote("dev3")
merge_request.downvote("dev4")
return merge_request
def test_simple_rejected(rejected_mr):
assert rejected_mr.status == MergeRequestStatus.REJECTED
def test_rejected_with_approvals(rejected_mr):
rejected_mr.upvote("dev2")
rejected_mr.upvote("dev3")
assert rejected_mr.status == MergeRequestStatus.REJECTED
def test_rejected_to_pending(rejected_mr):
rejected_mr.upvote("dev1")
assert rejected_mr.status == MergeRequestStatus.PENDING
def test_rejected_to_approved(rejected_mr):
rejected_mr.upvote("dev1")
rejected_mr.upvote("dev2")
assert rejected_mr.status == MergeRequestStatus.APPROVED
[ 235 ]Unit Testing and Refactoring
Chapter 8
Remember that tests affect the main code as well, so the principles of clean code apply to
them as well. In this case, the Don't Repeat Yourself (DRY) principle that we explored in
previous chapters appears once again, and we can achieve it with the help of pytest
fixtures.
Besides creating multiple objects or exposing data that will be used throughout the test
suite, it's also possible to use them to set up some conditions, for example, to globally patch
some functions that we don't want to be called, or when we want patch objects to be used
instead.
Code coverage
Tests runners support coverage plugins (to be installed via pip ) that will provide useful
information about what lines in the code have been executed while the tests were running.
This information is of great help so that we know which parts of the code need to be
covered by tests, as well identifying improvements to be made (both in the production code
and in the tests). One of the most widely used libraries for this is coverage ( https:/​ / ​ pypi.
org/​ project/​ coverage/​ ).
While they are of great help (and we highly recommend that you use them and configure
your project to run coverage in the CI when tests are run), they can also be misleading;
particularly in Python, we can get a false impression if we don't pay close attention to the
coverage report.
Setting up rest coverage
In the case of pytest, we have to install the pytest-cov package (at the time of this
writing, version 2.5.1 is used in this book). Once installed, when the tests are run, we have
to tell the pytest runner that pytest-cov will also run, and which package (or packages)
should be covered (among other parameters and configurations).
This package supports multiple configurations, like different sorts of output formats, and
it's easy to integrate it with any CI tool, but among all these features a highly recommended
option is to set the flag that will tell us which lines haven't been covered by tests yet,
because this is what's going to help us diagnose our code and allow us to start writing more
tests.
[ 236 ]Unit Testing and Refactoring
Chapter 8
To show you an example of what this would look like, use the following command:
pytest \
--cov-report term-missing \
--cov=coverage_1 \
test_coverage_1.py
This will produce an output similar to the following:
test_coverage_1.py ................ [100%]
----------- coverage: platform linux, python 3.6.5-final-0 -----------
Name
Stmts Miss Cover Missing
---------------------------------------------
coverage_1.py 38
1 97%
53
Here, it's telling us that there is a line that doesn't have unit tests so that we can take a look
and see how to write a unit test for it. This is a common scenario where we realize that to
cover those missing lines, we need to refactor the code by creating smaller methods. As a
result, our code will look much better, as in the example we saw at the beginning of this
chapter.
The problem lies in the inverse situation—can we trust the high coverage? Does it mean our
code is correct? Unfortunately, having good test coverage is necessary but in sufficient
condition for clean code. Not having tests for parts of the code is clearly something bad.
Having tests is actually very good (and we can say this for the tests that do exist), and
actually asserts real conditions that they are a guarantee of quality for that part of the code.
However, we cannot say that is all that is required; despite having a high level of coverage,
even more tests are required.
These are the caveats of test coverage, which we will mention in the next section.
Caveats of test coverage
Python is interpreted and, at a very high-level, coverage tools take advantage of this to
identify the lines that were interpreted (run) while the tests were running. It will then
report this at the end. The fact that a line was interpreted does not mean that it was
properly tested, and this is why we should be careful about reading the final coverage
report and trusting what it says.
[ 237 ]Unit Testing and Refactoring
Chapter 8
This is actually true for any language. The fact that a line was exercised does not mean at all
that it was stressed with all its possible combinations. The fact that all branches run
successfully with the provided data only means that the code supported that combination,
but it doesn't tell us anything about any other possible combinations of parameters that
would make the program crash.
Use coverage as a tool to find blind spots in the code, but not as a metric
or target goal.
Mock objects
There are cases where our code is not the only thing that will be present in the context of
our tests. After all, the systems we design and build have to do something real, and that
usually means connecting to external services (databases, storage services, external APIs,
cloud services, and so on). Because they need to have those side-effects, they're inevitable.
As much as we abstract our code, program towards interfaces, and isolate code from
external factors in order to minimize side-effects, they will be present in our tests, and we
need an effective way to handle that.
Mock objects are one of the best tactics to defend against undesired side-effects. Our code
might need to perform an HTTP request or send a notification email, but we surely don't
want that to happen in our unit tests. Besides, unit tests should run quickly, as we want to
run them quite often (all the time, actually), and this means we cannot afford latency.
Therefore, real unit tests don't use any actual service—they don't connect to any database,
they don't issue HTTP requests, and basically, they do nothing other than exercise the logic
of the production code.
We need tests that do such things, but they aren't units. Integration tests are supposed to
test functionality with a broader perspective, almost mimicking the behavior of a user. But
they aren't fast. Because they connect to external systems and services, they take longer to
run and are more expensive. In general, we would like to have lots of unit tests that run
really quickly in order to run them all the time, and have integration tests run less often (for
instance, on any new merge request).
While mock objects are useful, abusing their use ranges between a code smell or an anti-
pattern is the first caveat we would like to mention before going into the details of it.
[ 238 ]Unit Testing and Refactoring
Chapter 8
A fair warning about patching and mocks
We said before that unit tests help us write better code, because the moment we want to
start testing parts of the code, we usually have to write them to be testable, which often
means they are also cohesive, granular, and small. These are all good traits to have in a
software component.
Another interesting gain is that testing will help us notice code smells in parts where we
thought our code was correct. One of the main warnings that our code has code smells is
whether we find ourselves trying to monkey-patch (or mock) a lot of different things just to
cover a simple test case.
The unittest module provides a tool for patching our objects at unittest.mock.patch .
Patching means that the original code (given by a string denoting its location at import
time), will be replaced by something else, other than its original code, being the default a
mock object. This replaces the code at run-time, and has the disadvantage that we are losing
contact with the original code that was there in the first place, making our tests a little more
shallow. It also carries performance considerations, because of the overhead that imposes
modifying objects in the interpreter at run-time, and it's something that might end up
update if we refactor our code and move things around.
Using monkey-patching or mocks in our tests might be acceptable, and by itself it doesn't
represent an issue. On the other hand, abuse in monkey-patching is indeed a flag that
something has to be improved in our code.
Using mock objects
In unit testing terminology, there are several types of object that fall into the category
named test doubles. A test double is a type of object that will take the place of a real one in
our test suite for different kinds of reasons (maybe we don't need the actual production
code, but just a dummy object would work, or maybe we can't use it because it requires
access to services or it has side-effects that we don't want in our unit tests, and so on).
There are different types of test double, such as dummy objects, stubs, spies, or mocks.
Mocks are the most general type of object, and since they're quite flexible and versatile, they
are appropriate for all cases without needing to go into much detail about the rest of them.
It is for this reason that the standard library also includes an object of this kind, and it is
common in most Python programs. That's the one we are going to be using
here: unittest.mock.Mock .
[ 239 ]Unit Testing and Refactoring
Chapter 8
A mock is a type of object created to a specification (usually resembling the object of a
production class) and some configured responses (that is, we can tell the mock what it
should return upon certain calls, and what its behavior should be). The Mock object will
then record, as part of its internal status, how it was called (with what parameters, how
many times, and so on), and we can use that information to verify the behavior of our
application at a later stage.
In the case of Python, the Mock object that's available from the standard library provides a
nice API to make all sorts of behavioral assertions, such as checking how many times the
mock was called, with what parameters, and so on.
Types of mocks
The standard library provides Mock and MagicMock objects in the unittest.mock
module. The former is a test double that can be configured to return any value and will
keep track of the calls that were made to it. The latter does the same, but it also supports
magic methods. This means that, if we have written idiomatic code that uses magic
methods (and parts of the code we are testing will rely on that), it's likely that we will have
to use a MagicMock instance instead of just a Mock .
Trying to use Mock when our code needs to call magic methods will result in an error. See
the following code for an example of this:
class GitBranch:
def __init__(self, commits: List[Dict]):
self._commits = {c["id"]: c for c in commits}
def __getitem__(self, commit_id):
return self._commits[commit_id]
def __len__(self):
return len(self._commits)
def author_by_id(commit_id, branch):
return branch[commit_id]["author"]
[ 240 ]Unit Testing and Refactoring
Chapter 8
We want to test this function; however, another test needs to call
the author_by_id function. For some reason, since we're not testing that function, any
value provided to that function (and returned) will be good:
def test_find_commit():
branch = GitBranch([{"id": "123", "author": "dev1"}])
assert author_by_id("123", branch) == "dev1"
def test_find_any():
author = author_by_id("123", Mock()) is not None
# ... rest of the tests..
As anticipated, this will not work:
def author_by_id(commit_id, branch):
> return branch[commit_id]["author"]
E TypeError: 'Mock' object is not subscriptable
Using MagicMock instead will work. We can even configure the magic method of this type
of mock to return something we need in order to control the execution of our test:
def test_find_any():
mbranch = MagicMock()
mbranch.__getitem__.return_value = {"author": "test"}
assert author_by_id("123", mbranch) == "test"
A use case for test doubles
To see a possible use of mocks, we need to add a new component to our application that
will be in charge of notifying the merge request of the status of the build . When a build
is finished, this object will be called with the ID of the merge request and the status of the
build, and it will update the status of the merge request with this information by
sending an HTTP POST request to a particular fixed endpoint:
# mock_2.py
from datetime import datetime
import requests
from constants import STATUS_ENDPOINT
class BuildStatus:
"""The CI status of a pull request."""
[ 241 ]Unit Testing and Refactoring
Chapter 8
@staticmethod
def build_date() -> str:
return datetime.utcnow().isoformat()
@classmethod
def notify(cls, merge_request_id, status):
build_status = {
"id": merge_request_id,
"status": status,
"built_at": cls.build_date(),
}
response = requests.post(STATUS_ENDPOINT, json=build_status)
response.raise_for_status()
return response
This class has many side-effects, but one of them is an important external dependency
which is hard to surmount. If we try to write a test over it without modifying anything, it
will fail with a connection error as soon as it tries to perform the HTTP connection.
As a testing goal, we just want to make sure that the information is composed correctly, and
that library requests are being called with the appropriate parameters. Since this is an
external dependency, we don't test requests; just checking that it's called correctly will be
enough.
Another problem we will face when trying to compare data being sent to the library is that
the class is calculating the current timestamp, which is impossible to predict in a unit test.
Patching datetime directly is not possible, because the module is written in C. There are
some external libraries that can do that ( freezegun, for example), but they come with a
performance penalty, and for this example would be overkill. Therefore, we opt to
wrapping the functionality we want in a static method that we will be able to patch.
Now that we have established the points that need to be replaced in the code, let's write the
unit test:
# test_mock_2.py
from unittest import mock
from constants import STATUS_ENDPOINT
from mock_2 import BuildStatus
@mock.patch("mock_2.requests")
def test_build_notification_sent(mock_requests):
build_date = "2018-01-01T00:00:01"
with mock.patch("mock_2.BuildStatus.build_date",
[ 242 ]Unit Testing and Refactoring
Chapter 8
return_value=build_date):
BuildStatus.notify(123, "OK")
expected_payload = {"id": 123, "status": "OK", "built_at":
build_date}
mock_requests.post.assert_called_with(
STATUS_ENDPOINT, json=expected_payload
)
First, we use mock.patch as a decorator to replace the requests module. The result of this
function will create a mock object that will be passed as a parameter to the test
(named mock_requests in this example). Then, we use this function again, but this time as
a context manager to change the return value of the method of the class that computes the
date of the build, replacing the value with one we control, that we will use in the assertion.
Once we have all of this in place, we can call the class method with some parameters, and
then we can use the mock object to check how it was called. In this case, we are using the
method to see if requests.post was indeed called with the parameters as we wanted
them to be composed.
This is a nice feature of mocks—not only do they put some boundaries around all external
components (in this case to prevent actually sending some notifications or issuing HTTP
requests), but they also provide a useful API to verify the calls and their parameters.
While, in this case, we were able to test the code by setting the respective mock objects in
place, it's also true that we had to patch quite a lot in proportion to the total lines of code for
the main functionality. There is no rule about the ratio of pure productive code being tested
versus how many parts of that code we have to mock, but certainly, by using common
sense, we can see that, if we had to patch quite a lot of things in the same parts, something
is not clearly abstracted, and it looks like a code smell.
In the next section, we will explore how to refactor code to overcome this issue.