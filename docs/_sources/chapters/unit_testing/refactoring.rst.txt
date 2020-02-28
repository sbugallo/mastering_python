3. Refactoring
**************

**Refactoring** is a critical activity in software maintenance, yet something that can't be done
(at least correctly) without having unit tests. Every now and then, we need to support a
new feature or use our software in unintended ways. We need to realize that the only way
to accommodate such requirements is by first refactoring our code, make it more generic.
Only then can we move forward.

Typically, when refactoring our code, we want to improve its structure and make it better,
sometimes more generic, more readable, or more flexible. The challenge is to achieve these
goals while at the same time preserving the exact same functionality it had prior to the
modifications that were made. This means that, in the eyes of the clients of those
components we're refactoring, it might as well be the case that nothing had happened at all.

This constraint of having to support the same functionalities as before but with a different
version of the code implies that we need to run regression tests on code that was modified.
The only cost-effective way of running regression tests is if those tests are automatic. The
most cost-effective version of automatic tests is unit tests.

3.1. Evolving our code
++++++++++++++++++++++

In the previous example, we were able to separate out the side-effects from our code to
make it testable by patching those parts of the code that depended on things we couldn't
control on the unit test. This is a good approach since, after all, the ``mock.patch`` function
comes in handy for these sorts of task and replaces the objects we tell it to, giving us back a
``Mock`` object.

The downside of that is that we have to provide the path of the object we are going to
mock, including the module, as a string. This is a bit fragile, because if we refactor our code
(let's say we rename the file or move it to some other location), all the places with the patch
will have to be updated, or the test will break.

In the example, the fact that the ``notify()`` method directly depends on an implementation
detail (the requests module) is a design issue, that is, it is taking its toll on the unit tests as
well with the aforementioned fragility that is implied.

We still need to replace those methods with doubles (mocks), but if we refactor the code,
we can do it in a better way. Let's separate these methods into smaller ones, and most
importantly inject the dependency rather than keep it fixed. The code now applies
the dependency inversion principle, and it expects to work with something that supports
an interface (in this example, implicit one) such as the one the requests module provides:

.. code-block:: python

    from datetime import datetime
    from constants import STATUS_ENDPOINT


    class BuildStatus:
        endpoint = STATUS_ENDPOINT

        def __init__(self, transport):
            self.transport = transport

        @staticmethod
        def build_date() -> str:
            return datetime.now().isoformat()

        def compose_payload(self, merge_request_id, status) -> dict:
            return {
                "id": merge_request_id,
                "status": status,
                "built_at": self.build_date(),
            }

        def deliver(self, payload):
            response = self.transport.post(self.endpoint, json=payload)
            response.raise_for_status()
            return response

        def notify(self, merge_request_id, status):
            return self.deliver(self.compose_payload(merge_request_id, status))

We separate the methods (not notify is now compose + deliver),
make ``compose_payload()`` a new method (so that we can replace, without the need to
patch the class), and require the transport dependency to be injected. Now that
transport is a dependency, it is much easier to change that object for any double we want.

It is even possible to expose a fixture of this object with the doubles replaced as required:

.. code-block:: python

    @pytest.fixture
    def build_status():
        bstatus = BuildStatus(Mock())
        bstatus.build_date = Mock(return_value="2018-01-01T00:00:01")
        return bstatus

    def test_build_notification_sent(build_status):
        build_status.notify(1234, "OK")
        expected_payload = {
            "id": 1234,
            "status": "OK",
            "built_at": build_status.build_date(),
        }

        build_status.transport.post.assert_called_with(build_status.endpoint, json=expected_payload)

3.2. Production code isn't the only thing that evolves
++++++++++++++++++++++++++++++++++++++++++++++++++++++

We keep saying that unit tests are as important as production code. And if we are careful
enough with production code as to create the best possible abstraction, why wouldn't we
do the same for unit tests?

If the code for unit tests is as important as the main code, then it's definitely wise to design
it with extensibility in mind and make it as maintainable as possible. After all, this is the
code that will have to be maintained by an engineer other than its original author, so it has
to be readable.

The reason why we pay so much attention to make the code's flexibility is that we know
requirements change and evolve over time, and eventually as domain business rules
change, our code will have to change as well to support these new requirements. Since the
production code changed to support new requirements, in turn, the testing code will have
to change as well to support the newer version of the production code.

In one of the first examples we used, we created a series of tests for the merge request
object, trying different combinations and checking the status at which the merge request
was left. This is a good first approach, but we can do better than that.

Once we understand the problem better, we can start creating better abstractions. With this,
the first idea that comes to mind is that we can create a higher-level abstraction that checks
for particular conditions. For example, if we have an object that is a test suite that
specifically targets the ``MergeRequest`` class, we know its functionality will be limited to the
behavior of this class (because it should comply to the SRP), and therefore we could create
specific testing methods on this testing class. These will only make sense for this class, but
that will be helpful in reducing a lot of boilerplate code.

Instead of repeating assertions that follow the exact same structure, we can create a method
that encapsulates this and reuse it across all of the tests:

.. code-block:: python

    class TestMergeRequestStatus(unittest.TestCase):

        def setUp(self):
            self.merge_request = MergeRequest()

        def assert_rejected(self):
            self.assertEqual(self.merge_request.status, MergeRequestStatus.REJECTED)

        def assert_pending(self):
            self.assertEqual(self.merge_request.status, MergeRequestStatus.PENDING)

        def assert_approved(self):
            self.assertEqual(self.merge_request.status, MergeRequestStatus.APPROVED)

        def test_simple_rejected(self):
            self.merge_request.downvote("maintainer")
            self.assert_rejected()

        def test_just_created_is_pending(self):
            self.assert_pending()

If something changes with how we check the status of a merge request (or let's say we want
to add extra checks), there is only one place (the ``assert_approved()`` method) that will
have to be modified. More importantly, by creating these higher-level abstractions, the code
that started as merely unit tests starts to evolve into what could end up being a testing
framework with its own API or domain language, making testing more declarative.