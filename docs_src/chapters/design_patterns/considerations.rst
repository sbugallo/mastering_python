1. Considerations for design patterns in Python
***********************************************

Object-oriented design patterns are ideas of software construction that appear in different
scenarios when we deal with models of the problem we're solving. Because they're high-
level ideas, it's hard to think of them as being tied to particular programming languages.
They are instead more general concepts about how objects will interact in the application.
Of course, they will have their implementation details, varying from language to language,
but that doesn't form the essence of a design pattern.

That's the theoretical aspect of a design pattern, the fact that it is an abstract idea that
expresses concepts about the layout of the objects in the solution. There are plenty of other
books and several other resources about object-oriented design, and design patterns in
particular, so we are going to focus on those implementation details for
Python.

Given the nature of Python, some of the classical design patterns aren't actually needed.
That means that Python already supports features that render those patterns invisible.
Some argue that they don't exist in Python, but keep in mind that invisible doesn't mean
non-existing. They are there, just embedded in Python itself, so it's likely that we won't
even notice them.

Others have a much simpler implementation, again thanks to the dynamic nature of the
language, and the rest of them are practically the same as they are in other platforms, with
small differences.

In any case, the important goal for achieving clean code in Python is knowing what
patterns to implement and how. That means recognizing some of the patterns that Python
already abstracts and how we can leverage them. For instance, it would be completely non-
Pythonic to try to implement the standard definition of the iterator pattern (as we would do
in different languages), because (as we have already covered) iteration is deeply embedded
in Python, and the fact that we can create objects that will directly work in a for loop
makes this the right way to proceed.

Something similar happens with some of the creational patterns. Classes are regular objects
in Python, and so are functions. As we have seen in several examples so far, they can be
passed around, decorated, reassigned, and so on. That means that whatever kind of
customization we would like to make to our objects, we can most likely do it without
needing any particular setup of factory classes. In addition, there is no special syntax for
creating objects in Python (no new keyword, for example). This is another reason why,
most of the time, a simple function call will just work as a factory.

Other patterns are still needed, and we will see how, with some small adaptations, we can
make them more Pythonic, taking full advantage of the features that the language provides
(magic methods or the standard library).

Out of all the patterns available, not all of them are equally frequent, nor useful, so we will
focus on the main ones, those that we would expect to see the most in our applications, and
we will do so by following a pragmatic approach.