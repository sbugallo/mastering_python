5. Dependency inversion
***********************

The **dependency inversion principle (DIP)** proposes an interesting design principle by
which we protect our code by making it independent of things that are fragile, volatile, or
out of our control. The idea of inverting dependencies is that our code should not adapt to
details or concrete implementations, but rather the other way around: we want to force
whatever implementation or detail to adapt to our code via a sort of API.

Abstractions have to be organized in such a way that they do not depend on details, but
rather the other way around: the details (concrete implementations) should depend on
abstractions.

Imagine that two objects in our design need to collaborate, A and B. A works with an
instance of B, but as it turns out, our module doesn't control B directly (it might be an
external library, or a module maintained by another team, and so on). If our code heavily
depends on B, when this changes the code will break. To prevent this, we have to invert the
dependency: make B have to adapt to A. This is done by presenting an interface and forcing
our code not to depend on the concrete implementation of B, but rather on the interface we
have defined. It is then B's responsibility to comply with that interface.

In line with the concepts explored in previous sections, abstractions also come in the form
of interfaces (or abstract base classes in Python).

In general, we could expect concrete implementations to change much more frequently
than abstract components. It is for this reason that we place abstractions (interfaces) as
flexibility points where we expect our system to change, be modified, or extended without
the abstraction itself having to be changed.

5.1. A case of rigid dependencies
+++++++++++++++++++++++++++++++++

The last part of our event's monitoring system is to deliver the identified events to a data
collector to be further analyzed. A naive implementation of such an idea would consist of
having an event streamer class that interacts with a data destination, for example, ``Syslog``:

.. figure:: ../../../../_static/images/ch4_dip_bad_diagram.png
   :width: 30%
   :align: center

However, this design is not very good, because we have a high-level class
(``EventStreamer``) depending on a low-level one (``Syslog`` is an implementation detail). If
something changes in the way we want to send data to ``Syslog``, ``EventStreamer`` will have
to be modified. If we want to change the data destination for a different one or add new
ones at runtime, we are also in trouble because we will find ourselves constantly modifying
the ``stream()`` method to adapt it to these requirements.

5.2. Inverting the dependencies
+++++++++++++++++++++++++++++++

The solution to these problems is to make ``EventStreamer`` work with an interface, rather
than a concrete class. This way, implementing this interface is up to the low-level classes
that contain the implementation details:

.. figure:: ../../../../_static/images/ch4_dip_good_diagram.png
   :width: 40%
   :align: center

Now there is an interface that represents a generic data target where data is going to be sent
to. Notice how the dependencies have now been inverted since ``EventStreamer`` does not
depend on a concrete implementation of a particular data target, it does not have to change
in line with changes on this one, and it is up to every particular data target; to implement
the interface correctly and adapt to changes if necessary.

In other words, the original ``EventStreamer`` of the first implementation only worked with
objects of type Syslog, which was not very flexible. Then we realized that it could work
with any object that could respond to a .send() message, and identified this method as the
interface that it needed to comply with. Now, in this version, Syslog is actually extending
the abstract base class named ``DataTargetClient``, which defines the ``send()`` method.

From now on, it is up to every new type of data target (email, for instance) to extend this
abstract base class and implement the ``send()`` method.

We can even modify this property at runtime for any other object that implements a
``send()`` method, and it will still work. This is the reason why it is often called dependency
injection: because the dependency can be provided dynamically.

The avid reader might be wondering why this is actually necessary. Python is flexible
enough (sometimes too flexible), and will allow us to provide an object like
``EventStreamer`` with any particular data target object, without this one having to comply
with any interface because it is dynamically typed. The question is this: why do we need to
define the abstract base class (interface) at all when we can simply pass an object with a
``send()`` method to it?

In all fairness, this is true; there is actually no need to do that, and the program will work
just the same. After all, polymorphism does not mean (or require) inheritance to work.
However, defining the abstract base class is a good practice that comes with some
advantages, the first one being duck typing. Together with as duck typing, we can mention
the fact that the models become more readable: remember that inheritance follows the rule
of is a, so by declaring the abstract base class and extending from it, we are saying that, for
instance, ``Syslog`` is ``DataTargetClient``, which is something users of your code can read
and understand (again, this is duck typing).

All in all, it is not mandatory to define the abstract base class, but it is desirable in order to
achieve a cleaner design.
