4. Final thoughts about design patterns
***************************************

We have seen the world of design patterns in Python, and in doing so, we have found
solutions to common problems, as well as more techniques that will help us achieve a clean
design.

All of this sounds good, but it begs the question, how good are design patterns? Some
people argue that they do more harm than good, that they were created for languages
whose limited type system (and lack of first-class functions) makes it impossible to
accomplish things we would normally do in Python. Others claim that design patterns
force a design solution, creating some bias that limits a design that would have otherwise
emerged, and which would have been better. Let's look at each of these points in turn.

4.1. The influence of patterns over the design
++++++++++++++++++++++++++++++++++++++++++++++

A design pattern, as with any other topic in software engineering, cannot be good or bad
in and of itself, but rather in how it's implemented. In some cases, there is actually no need
for a design pattern, and a simpler solution would do. Trying to force a pattern where it
doesn't fit is a case of over-engineering, and that's clearly bad, but it doesn't mean that there
is a problem with the design patterns, and most likely in these scenarios, the problem is not
even related to patterns at all. Some people try to over-engineer everything because they
don't understand what flexible and adaptable software really means. As we mentioned
before in this book, making good software is not about anticipating future requirements
(there is no point in doing futurology), but just solving the problem that we have at
hand right now, in a way that doesn't prevent us from making changes to it in the future. It
doesn't have to handle those changes now; it just needs to be flexible enough so that it can
be modified in the future. And when that future comes, we will still have to remember the
rule of three or more instances of the same problem before coming up with a generic
solution or a proper abstraction.

This is typically the point where the design patterns should emerge, once we have
identified the problem correctly and are able to recognize the pattern and abstract
accordingly.

Let's come back to the topic of the suitability of the patterns to the language. As we said in
the introduction of the chapter, design patterns are high-level ideas. They typically refer to
the relation of objects and their interactions. It's hard to think that such things might
disappear from one language to another. It's true that some patterns are actually
implemented manually in Python, as is the case of the iterator pattern (which, as it was
heavily discussed earlier in the book, is built in Python), or a strategy (because, instead, we
would just pass functions as any other regular object; we don't need to encapsulate the
strategy method into an object the function itself would be an object).

But other patterns are actually needed, and they indeed solve problems, as in the case of the
decorator and composite patterns. In other cases, there are design patterns that Python
itself implements, and we just don't always see them, as in the case of the facade pattern
that we discussed in the section on ``os``.

As to our design patterns leading our solution in a wrong direction, we have to be careful
here. Once again, it's better if we start designing our solution by thinking in terms of the
domain problem and creating the right abstractions, and then later see whether there is a
design pattern that emerges from that design. Let's say that it does. Is that a bad thing? The
fact that there is already a solution to the problem we're trying to solve cannot be a bad
thing. It would be bad to reinvent the wheel, as happens many times in our field. Moreover,
the fact that we are applying a pattern, something already proven and validated, should
give us greater confidence in the quality of what we are building.

4.2. Names in our models
++++++++++++++++++++++++

Should we mention that we are using a design pattern in our code?

If the design is good and the code is clean, it should speak for itself. It is not recommended
that you name things after the design patterns you are using for a couple of reasons:

- Users of our code and other developers don't need to know the design pattern behind the code, as long as it works as intended.
- Stating the design pattern ruins the intention revealing principle. Adding the name of the design pattern to a class makes it lose part of its original meaning. If a class represents a query, it should be named ``Query`` or ``EnhancedQuery``, something that reveals the intention of what that object is supposed to do. ``EnhancedQueryDecorator`` doesn't mean anything meaningful, and the ``Decorator`` suffix creates more confusion than clarity.

Mentioning the design patterns in docstrings might be acceptable because they work as
documentation, and expressing the design ideas (again, communicating) in our design is a
good thing. However, this should not be needed. Most of the time, though, we do not need
to know that a design pattern is there.

The best designs are those in which design patterns are completely transparent to the users.
An example of this is how the facade pattern appears in the standard library, making it
completely transparent to users as to how to access the os module. An even more elegant
example is how the iterator design pattern is so completely abstracted by the language that
we don't even have to think about it.
