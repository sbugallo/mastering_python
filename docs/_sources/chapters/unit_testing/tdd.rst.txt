5. A brief introduction to test-driven development
**************************************************

There are entire books dedicated only to TDD, so it would not be realistic to try and cover
this topic comprehensively. However, it's such an important topic that it has to
be mentioned.

The idea behind TDD is that tests should be written before production code in a way that
the production code is only written to respond to tests that are failing due to that missing
implementation of the functionality.

There are multiple reasons why we would like to write the tests first and then the code.
From a pragmatic point of view, we would be covering our production code quite
accurately. Since all of the production code was written to respond to a unit test, it would
be highly unlikely that there are tests missing for functionality (that doesn't mean that there
is 100% of coverage of course, but at least all main functions, methods, or components will
have their respective tests, even if they aren't completely covered).

The workflow is simple and at a high-level consist of three steps. First, we write a unit test
that describes something we need to be implemented. When we run this test, it will fail,
because that functionality has not been implemented yet. Then, we move onto
implementing the minimal required code that satisfies that condition, and we run the test
again. This time, the test should pass. Now, we can improve (refactor) the code.

This cycle has been popularized as the famous **red-green-refactor**, meaning that in the
beginning, the tests fail (red), then we make them pass (green), and then we proceed to
refactor the code and iterate it.
