4. Interface segregation
************************

The **interface segregation principle (ISP)** provides some guidelines over an idea that we
have revisited quite repeatedly already: that interfaces should be small.

In object-oriented terms, an interface is represented by the set of methods an object
exposes. This is to say that all the messages that an object is able to receive or interpret
constitute its interface, and this is what other clients can request. The interface separates the
definition of the exposed behavior for a class from its implementation.

In Python, interfaces are implicitly defined by a class according to its methods. This is
because Python follows the so-called **duck typing** principle.

Traditionally, the idea behind duck typing was that any object is really represented by the
methods it has, and by what it is capable of doing. This means that, regardless of the type of
the class, its name, its docstring, class attributes, or instance attributes, what ultimately
defines the essence of the object are the methods it has. The methods defined on a class
(what it knows how to do) are what determines what that object will actually be. It was
called duck typing because of the idea that "If it walks like a duck, and quacks like a duck,
it must be a duck."

For a long time, duck typing was the sole way interfaces were defined in Python. Later on,
Python 3 (PEP-3119) introduced the concept of abstract base classes as a way to define
interfaces in a different way. The basic idea of abstract base classes is that they define a
basic behavior or interface that some derived classes are responsible for implementing. This
is useful in situations where we want to make sure that certain critical methods are actually
overridden, and it also works as a mechanism for overriding or extending the functionality
of methods such as ``isinstance()``.

This module also contains a way of registering some types as part of a hierarchy, in what is
called a **virtual subclass**. The idea is that this extends the concept of duck typing a little bit
further by adding a new criterion—walks like a duck, quacks like a duck, or... it says it is a
duck.

These notions of how Python interprets interfaces are important for understanding this
principle and the next one.

In abstract terms, this means that the ISP states that, when we define an interface that
provides multiple methods, it is better to instead break it down into multiple ones, each one
containing fewer methods (preferably just one), with a very specific and accurate scope. By
separating interfaces into the smallest possible units, to favor code reusability, each class
that wants to implement one of these interfaces will most likely be highly cohesive given
that it has a quite definite behavior and set of responsibilities.

4.1. An interface that provides too much
++++++++++++++++++++++++++++++++++++++++

Now, we want to be able to parse an event from several data sources, in different formats
(XML and JSON, for instance). Following good practice, we decide to target an interface as
our dependency instead of a concrete class, and something like the following is devised:

.. figure:: ../../_static/images/ch4_isp_bad_interface.png
   :width: 20%
   :align: center

In order to create this as an interface in Python, we would use an abstract base class and
define the methods (``from_xml()`` and ``from_json()``) as abstract, to force derived classes to
implement them. Events that derive from this abstract base class and implement these
methods would be able to work with their corresponding types.

But what if a particular class does not need the XML method, and can only be constructed
from a JSON? It would still carry the from_xml() method from the interface, and since it
does not need it, it will have to pass. This is not very flexible as it creates coupling and
forces clients of the interface to work with methods that they do not need.

4.2. The smaller the interface, the better
++++++++++++++++++++++++++++++++++++++++++

It would be better to separate this into two different interfaces, one for each method:

.. figure:: ../../_static/images/ch4_isp_good_interface.png
   :width: 40%
   :align: center

With this design, objects that derive from ``XMLEventParser`` and implement the
``from_xml()`` method will know how to be constructed from an XML, and the same for a
JSON file, but most importantly, we maintain the orthogonality of two independent
functions, and preserve the flexibility of the system without losing any functionality that
can still be achieved by composing new smaller objects.

There is some resemblance to the SRP, but the main difference is that here we are talking
about interfaces, so it is an abstract definition of behavior. There is no reason to change
because there is nothing there until the interface is actually implemented. However, failure
to comply with this principle will create an interface that will be coupled with orthogonal
functionality, and this derived class will also fail to comply with the SRP (it will have more
than one reason to change).

4.3. How small should an interface be?
++++++++++++++++++++++++++++++++++++++

The point made in the previous section is valid, but it also needs a warning: avoid a
dangerous path if it's misunderstood or taken to the extreme.

A base class (abstract or not) defines an interface for all the other classes to extend it. The
fact that this should be as small as possible has to be understood in terms of cohesion: it
should do one thing. That doesn't mean it must necessarily have one method. In the
previous example, it was by coincidence that both methods were doing totally disjoint
things, hence it made sense to separate them into different classes.

But it could be the case that more than one method rightfully belongs to the same class.
Imagine that you want to provide a mixin class that abstracts certain logic in a context
manager so that all classes derived from that mixin gain that context manager logic for free.
As we already know, a context manager entails two methods: ``__enter__`` and ``__exit__``.
They must go together, or the outcome will not be a valid context manager at all!

Failure to place both methods in the same class will result in a broken component that is
not only useless, but also misleadingly dangerous. Hopefully, this exaggerated example
works as a counter-balance to the one in the previous section, and together the reader can
get a more accurate picture about designing interfaces.
