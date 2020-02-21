4. More about unit testing
**************************

With the concepts we have revisited so far, we know how to test our code, think about our
design in terms of how it is going to be tested, and configure the tools in our project to run
the automated tests that will give us some degree of confidence over the quality of the
software we have written.
[ 247 ]Unit Testing and Refactoring
Chapter 8
If our confidence in the code is determined by the unit tests written on it, how do we know
that they are enough? How could we be sure that we have been through enough on the test
scenarios and that we are not missing some tests? Who says that these tests are correct?
Meaning, who tests the tests?
The first part of the question, about being thorough on the tests we wrote, is answered by
going beyond in our testing efforts through property-based testing.
The second part of the question might have multiple answers from different points of view,
but we are going to briefly mention mutation testing as a means of determining that our
tests are indeed correct. In this sense, we are thinking that the unit tests check our main
productive code, and this works as a control for the unit tests as well.
Property-based testing
Property-based testing consists of generating data for tests cases with the goal of finding
scenarios that will make the code fail, which weren't covered by our previous unit tests.
The main library for this is hypothesis which, configured along with our unit tests, will
help us find problematic data that will make our code fail.
We can imagine that what this library does is find counter examples for our code. We write
our production code (and unit tests for it!), and we claim it's correct. Now, with this library,
we define some hypothesis that must hold for our code, and if there are some cases where
our assertions don't hold, the hypothesis will provide a set of data that causes the error.
The best thing about unit tests is that they make us think harder about our production code.
The best thing about the hypothesis is that it makes us think harder about our unit tests.
Mutation testing
We know that tests are the formal verification method we have to ensure that our code is
correct. And what makes sure that the test is correct? The production code, you might
think, and yes, in a way this is correct, we can think of the main code as a counter balance
for our tests.
[ 248 ]Unit Testing and Refactoring
Chapter 8
The point in writing unit tests is that we are protecting ourselves against bugs, and testing
for failure scenarios we really don't want to happen in production. It's good that the tests
pass, but it would be bad if they pass for the wrong reasons. That is, we can use unit tests as
an automatic regression tool—if someone introduces a bug in the code, later on, we expect
at least one of our tests to catch it and fail. If this doesn't happen, either there is a test
missing, or the ones we had are not doing the right checks.
This is the idea behind mutation testing. With a mutation testing tool, the code will be
modified to new versions (called mutants), that are variations of the original code but with
some of its logic altered (for example, operators are swapped, conditions are inverted, and
so on). A good test suite should catch these mutants and kill them, in which case it means
we can rely on the tests. If some mutants survive the experiment, it's usually a bad sign. Of
course, this is not entirely precise, so there are intermediate states we might want to ignore.
To quickly show you how this works and to allow you to get a practical idea of this, we are
going to use a different version of the code that computes the status of a merge request
based on the number of approvals and rejections. This time, we have changed the code for a
simple version that, based on these numbers, returns the result. We have moved the
enumeration with the constants for the statuses to a separate module so that it now looks
more compact:
# File mutation_testing_1.py
from mrstatus import MergeRequestStatus as Status
def evaluate_merge_request(upvote_count, downvotes_count):
if downvotes_count > 0:
return Status.REJECTED
if upvote_count >= 2:
return Status.APPROVED
return Status.PENDING
And now will we add a simple unit test, checking one of the conditions and its expected
result :
# file: test_mutation_testing_1.py
class TestMergeRequestEvaluation(unittest.TestCase):
def test_approved(self):
result = evaluate_merge_request(3, 0)
self.assertEqual(result, Status.APPROVED)
Now, we will install mutpy, a mutation testing tool for Python, with pip install mutpy,
and tell it to run the mutation testing for this module with these tests:
$ mut.py \
--target mutation_testing_$N \
[ 249 ]Unit Testing and Refactoring
Chapter 8
--unit-test test_mutation_testing_$N \
--operator AOD `# delete arithmetic operator` \
--operator AOR `# replace arithmetic operator` \
--operator COD `# delete conditional operator` \
--operator COI `# insert conditional operator` \
--operator CRP `# replace constant` \
--operator ROR `# replace relational operator` \
--show-mutants
The result is going to look something similar to this:
[*] Mutation score [0.04649 s]: 100.0%
- all: 4
- killed: 4 (100.0%)
- survived: 0 (0.0%)
- incompetent: 0 (0.0%)
- timeout: 0 (0.0%)
This is a good sign. Let's take a particular instance to analyze what happened. One of the
lines on the output shows the following mutant:
- [# 1] ROR mutation_testing_1:11 :
------------------------------------------------------
7: from mrstatus import MergeRequestStatus as Status
8:
9:
10: def evaluate_merge_request(upvote_count, downvotes_count):
~11:
if downvotes_count < 0:
12:
return Status.REJECTED
13:
if upvote_count >= 2:
14:
return Status.APPROVED
15:
return Status.PENDING
------------------------------------------------------
[0.00401 s] killed by test_approved
(test_mutation_testing_1.TestMergeRequestEvaluation)
Notice that this mutant consists of the original version with the operator changed in line 11
( > for < ), and the result is telling us that this mutant was killed by the tests. This means that
with this version of the code (let's imagine that someone by mistakes makes this change),
then the result of the function would have been APPROVED, and since the test expects it to
be REJECTED, it fails, which is a good sign (the test caught the bug that was introduced).
[ 250 ]Unit Testing and Refactoring
Chapter 8
Mutation testing is a good way to assure the quality of the unit tests, but it requires some
effort and careful analysis. By using this tool in complex environments, we will have to take
some time analyzing each scenario. It is also true that it is expensive to run these tests
because it requires multiples runs of different versions of the code, which might take up too
many resources and may take longer to complete. However, it would be even more
expensive to have to make these checks manually and will require much more effort. Not
doing these checks at all might be even riskier, because we would be jeopardizing the
quality of the tests.