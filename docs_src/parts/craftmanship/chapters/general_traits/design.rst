1. Design by contract
*********************

Some parts of the software we are working on are not meant to be called directly by users, but instead by
other parts of the code. Such is the case when we divide the responsibilities of the application into
different components or layers, and we have to think about the interaction between them.

We will have to encapsulate some functionality behind each component, and expose an interface to clients who
are going to use that functionality, namely an **Application Programming Interface (API)**. The functions,
classes, or methods we write for that component have a particular way of working under certain considerations
that, if they are not met, will make our code crash. Conversely, clients calling that code expect a particular
response, and any failure of our function to provide this would represent a defect. That is to say that if,
for example, we have a function that is expected to work with a series of parameters of type integers, and
some other function invokes our passing strings, it is clear that it should not work as expected, but in
reality, the function should not run at all because it was called incorrectly (the client made a mistake).
This error should not pass silently.

Of course, when designing an API, the expected input, output, and side-effects should be documented. But
documentation cannot enforce the behavior of the software at runtime. These rules, what every part of the code
expects in order to work properly and what the caller is expecting from them, should be part of the design,
and here is where the concept of a contract comes into place.

The idea behind the DbC is that instead of implicitly placing in the code what every party is expecting, both
parties agree on a contract that, if violated, will raise an exception, clearly stating why it cannot
continue.

In our context, a contract is a construction that enforces some rules that must be honored during the
communication of software components. A contract entails mainly preconditions and postconditions, but in some
cases, invariants, and side-effects are also described:

- **Preconditions**: We can say that these are all the checks code will do before running. It will check for all the conditions that have to be made before the function can proceed. In general, it's implemented by validating the data set provided in the parameters passed, but nothing should stop us from running all sorts of validations (for example, validating a set in a database, a file, another method that was called before, and so on) if we consider that their side-effects are overshadowed by the importance of such a validation. Notice that this imposes a constraint on the caller.
- **Postconditions**: The opposite of preconditions, here, the validations are done after the function call is returned. Postcondition validations are run to validate what the caller is expecting from this component.
- **Invariants**: Optionally, it would be a good idea to document, in the docstring of a function, the invariants, the things that are kept constant while the code of the function is running, as an expression of the logic of the function to be correct.
- **Side-effects**: Optionally, we can mention any side-effects of our code in the docstring.

While conceptually all of these items form part of the contract for a software component, and this is what
should go to the documentation of such piece, only the first two (preconditions and postconditions) are to be
enforced at a low level (code).

The reason why we would design by contract is that if errors occur, they must
be easy to spot (and by noticing whether it was either the precondition or postcondition that failed, we will
find the culprit much easily) so that they can be quickly corrected. More importantly, we want critical parts
of the code to avoid being executed under the wrong assumptions. This should help to clearly mark the limits
for the responsibilities and errors if they occur, as opposed to something saying—this part of the application
is failing... But the caller code provided the wrong arguments, so where should we apply the fix?

The idea is that preconditions bind the client (they have an obligation to meet them if they want to run some part of the
code), whereas postconditions bind the component in question to some guarantees that the client can verify and
enforce.

This way, we can quickly identify responsibilities. If the precondition fails, we know it is due to a
defect on the client. On the other hand, if the postcondition check fails, we know the problem is in the
routine or class (supplier) itself.

Specifically regarding preconditions, it is important to highlight that they can be checked at runtime, and if
they occur, the code that is being called should not be run at all (it does not make sense to run it because
its conditions do not hold, and further more, doing so might end up making things worse).

1.1. Preconditions
++++++++++++++++++

Preconditions are all of the guarantees a function or method expects to receive in order to work correctly. In
general programming terms, this usually means to provide data that is properly formed, for example, objects
that are initialized, non-null values, and many more. For Python, in particular, being dynamically typed, this
also means that sometimes we need to check for the exact type of data that is provided. This is not exactly
the same as type checking, the kind mypy would do this, but rather verify for exact values that are needed.

Part of these checks can be detected early on by using static analysis tools, such as mypy, but
these checks are not enough. A function should have proper validation for the information that it is going to
handle.

Now, this poses the question of where to place the validation logic, depending on whether we let the clients
validate all the data before calling the function, or allow this one to validate everything that it received
prior running its own logic. The former corresponds to a tolerant approach (because the function itself is
still allowing any data, potentially malformed data as well), whereas the latter corresponds to a demanding
approach.

For the purposes of this analysis, we prefer a demanding approach when it comes to DbC, because it is usually
the safest choice in terms of robustness, and usually the most common practice in the industry.

Regardless of the approach we decide to take, we should always keep in mind the non-redundancy principle,
which states that the enforcement of each precondition for a function should be done by only one of the two
parts of the contract, but not both. This means that we put the validation logic on the client, or we leave it
to the function itself, but in no cases should we duplicate it (which also relates to the DRY principle).

1.2. Postconditions
+++++++++++++++++++

Postconditions are the part of the contract that is responsible for enforcing the state after the method or
function has returned.

Assuming that the function or method has been called with the correct properties (that is, with its
preconditions met), then the postconditions will guarantee that certain properties are preserved.

The idea is to use postconditions to check and validate for everything that a client might need. If the method
executed properly, and the postcondition validations pass, then any client calling that code should be able to
work with the returned object without problems, as the contract has been fulfilled.

1.3. Pythonic contracts
+++++++++++++++++++++++

Programming by Contract for Python, is deferred. This doesn't mean that we cannot implement it in Python,
because, as introduced at the beginningf, this is a general design principle.

Probably the best way to enforce this is by adding control mechanisms to our methods, functions, and classes,
and if they fail raise a RuntimeError exception or ValueError . It's hard to devise a general rule for the
correct type of exception, as that would pretty much depend on the application in particular. These previously
mentioned exceptions are the most common types of exception, but if they don't fit accurately with the
problem, creating a custom exception would be the best choice.

We would also like to keep the code as isolated as possible. That is, the code for the preconditions in one
part, the one for the postconditions in another, and the core of the function separated. We could achieve this
separation by creating smaller functions, but in some cases implementing a decorator would be an interesting
alternative.

1.4. Conclusions
++++++++++++++++

The main value of this design principle is to effectively identify where the problem is. By defining a
contract, when something fails at runtime it will be clear what part of the code is broken, and what broke the
contract.

As a result of following this principle, the code will be more robust. Each component is enforcing its own
constraints and maintaining some invariants, and the program can be proven correct as long as these invariants
are preserved.

It also serves the purpose of clarifying the structure of the program better. Instead of trying to run ad hoc
validations, or trying to surmount all possible failure scenarios, the contracts explicitly specify what each
function or method expects to work properly, and what is expected from them.

Of course, following these principles also adds extra work, because we are not just programming the core logic
of our main application, but also the contracts. In addition, we might also want to consider adding unit tests
for these contracts as well. However, the quality gained by this approach pays off in the long run; hence, it
is a good idea to implement this principle for critical components of the application.

Nonetheless, for this method to be effective, we should carefully think about what are we willing to validate,
and this has to be a meaningful value. For example, it would not make much sense to define contracts that only
check for the correct data types of the parameters provided to a function. Many programmers would argue that
this would be like trying to make Python a statically-typed language. Regardless of this, tools such as Mypy,
in combination with the use of annotations, would serve this purpose much better and with less effort. With
that in mind, design contracts so that there is actually value on them.
