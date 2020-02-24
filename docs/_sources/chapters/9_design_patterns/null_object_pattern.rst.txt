3. The null object pattern
**************************
The null object pattern is an idea that relates to the good practices that were mentioned in
previous chapters of this book. Here, we are formalizing them, and giving more context
and analysis to this idea.
The principle is rather simple—functions or methods must return objects of a consistent
type. If this is guaranteed, then clients of our code can use the objects that are returned with
polymorphism, without having to run extra checks on them.
In the previous examples, we explored how the dynamic nature of Python made things
easier for most design patterns. In some cases, they disappear entirely, and in others, they
are much easier to implement. The main goal of design patterns as they were originally
thought of is that methods or functions should not explicitly name the class of the object
that they need in order to work. For this reason, they propose the creation of interfaces and
a way of rearranging the objects to make them fit these interfaces in order to modify the
design. But most of the time, this is not needed in Python, and we can just pass different
objects, and as long as they respect the methods they must have, then the solution will
work.
On the other hand, the fact that objects don't necessarily have to comply with an interface
requires us to be more careful as to the things that are returning from such methods and
functions. In the same way that our functions didn't make any assumptions about what
they were receiving, it's fair to assume that clients of our code will not make any
assumptions either (it is our responsibility to provide objects that are compatible). This can
be enforced or validated with design by contract. Here, we will explore a simple pattern
that will help us avoid these kinds of problems.
Consider the chain or responsibility design pattern explored in the previous section. We
saw how flexible it is and its many advantages, such as decoupling responsibilities into
smaller objects. One of the problems it has is that we never actually know what object will
end up processing the message, if any. In particular, in our example, if there was no
suitable object to process the log line, then the method would simply return None .
We don't know how users will use the data we passed, but we do know that they are
expecting a dictionary. Therefore, the following error might occur:
AttributeError: 'NoneType' object has no attribute 'keys'
In this case, the fix is rather simple—the default value of the process() method should be
an empty dictionary rather than None .
Ensure that you return objects of a consistent type.
[ 280 ]Common Design Patterns
Chapter 9
But what if the method didn't return a dictionary, but a custom object of our domain?
To solve this problem, we should have a class that represents the empty state for that object
and return it. If we have a class that represents users in our system, and a function that
queries users by their ID, then in the case that a user is not found, it should do one of the
following two things:
Raise an exception
Return an object of the UserUnknown type
But in no case should it return None . The phrase None doesn't represent what just
happened, and the caller might legitimately try to ask methods to it, and it will fail with
AttributeError .
We have discussed exceptions and their pros and cons earlier on, so we should mention
that this null object should just have the same methods as the original user and do nothing
for each one of them.
The advantage of using this structure is that not only are we avoiding an error at runtime
but also that this object might be useful. It could make the code easier to test, and it can
even, for instance, help in debugging (maybe we could put logging into the methods to
understand why that state was reached, what data was provided to it, and so on).
By exploiting almost all of the magic methods of Python, it would be possible to create a
generic null object that does absolutely nothing, no matter how it is called, but which can
be called from almost any client. Such an object would slightly resemble a Mock object. It is
not advisable to go down that path because of the following reasons:
It loses meaning with the domain problem. Back in our example, having an object
of the UnknownUser type makes sense, and gives the caller a clear idea that
something went wrong with the query.
It doesn't respect the original interface. This is problematic. Remember that the
point is that an UnknownUser is a user, and therefore it must have the same
methods. If the caller accidentally asks for a method that is not there, then, in that
case, it should raise an AttributeError exception, and that would be good.
With the generic null object that can do anything and respond to anything, we
would be losing this information, and bugs might creep in. If we opt for creating
a Mock object with spec=User , then this anomaly would be caught, but again,
using a Mock object to represent what is actually an empty state harms
the intention revealing the degree of the code.
This pattern is a good practice that allows us to maintain polymorphism in our objects.
