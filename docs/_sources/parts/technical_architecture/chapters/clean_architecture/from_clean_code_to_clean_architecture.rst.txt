1. From clean code to clean architecture
****************************************

This section is a discussion of how concepts that were emphasized in previous chapters
reappear in a slightly different shape when we consider aspects of large systems. There is
an interesting resemblance to how concepts that apply to more detailed design, as well as
code, also apply to large systems and architectures.

The concepts explored in previously were related to single applications, generally, a
project, that might be a single repository (or a few), for a source control version system (git).
This is not to say that those design ideas are only applicable to code, or that they are of no
use when thinking of an architecture, for two reasons: the code is the foundation of the
architecture, and, if it's not written carefully, the system will fail regardless of how well
thought-out the architecture is.

Second, some principles that were revisited in previous chapters do not apply to code but
are instead design ideas. The clearest example comes from design patterns. They are high-
level ideas. With this, we can get a quick picture of how a component in our architecture
might appear, without going into the details of the code.

But large enterprise systems typically consist of many of these applications, and now it's
time to start thinking in terms of a larger design, in the form of a distributed system.

In the following sections, we discuss the main topics that have been already discussed, but now from the
perspective of a system.

1.1. Separation of concerns
+++++++++++++++++++++++++++

Inside an application, there are multiple components. Their code is divided into other
subcomponents, such as modules or packages, and the modules into classes or functions,
and the classes into methods. The emphasis has been on keeping
these components as small as possible, particularly in the case of functions, they
should do one thing, and be small.

Several reasons were presented to justify this rationale. Small functions are easier to
understand, follow, and debug. They are also easier to test. The smaller the pieces in our
code, the easier it will be to write unit tests for it.

For the components of each application, we wanted different traits, mainly high cohesion,
and low coupling. By dividing components into smaller units, each one with a single and
well-defined responsibility, we achieve a better structure where changes are easier to
manage. In the face of new requirements, there will be a single rightful place to make the
changes, and the rest of the code should probably be unaffected.

When we talk about code, we say component to refer to one of these cohesive units (it might
be a class, for example). When speaking in terms of an architecture, a component means
anything in the system that can be treated as a working unit. The term component itself is
quite vague, so there is no universally accepted definition in software architecture of what
this means more concretely. The concept of a working unit is something that can vary from
project to project. A component should be able to be released or deployed with its own
cycles, independently from the rest of the parts of the system. And it is precisely that, one of
the parts of a system, is namely the entire application.

For Python projects, a component could be a package, but a service can also be a
component. Notice how two different concepts, with different levels of granularity, can be
considered under the same category. To give an example, the event systems we used in
previous chapters could be considered a component. It's a working unit with a clearly
defined purpose (to enrich events identified from logs), it can be deployed independently
from the rest (whether as a Python package, or, if we expose its functionality, as a service),
and it's a part of the entire system, but not the whole application itself.

On the examples of previous chapters we have seen an idiomatic code, and we have also
highlighted the importance of good design for our code, with objects that have single well-
defined responsibilities, being isolated, orthogonal, and easier to maintain. This very same
criteria, which applies to detailed design (functions, classes, methods), also applies to the
components of a software architecture.

It's probably undesirable for a large system to be just one component. A monolithic
application will act as the single source of truth, responsible for everything in the system,
and that will carry a lot of undesired consequences (harder to isolate and identify changes,
to test effectively, and so on). In the same way, our code will be harder to maintain, if we
are not careful and place everything in one place, the application will suffer from similar
problems if its components aren't treated with the same level of attention.

The idea of creating cohesive components in a system can have more than one
implementation, depending on the level of abstraction we require.

One option would be to identify common logic that is likely to be reused multiple times
and place it in a Python package (we will discuss the details later in the chapter).
Another alternative would be to break the application into multiple smaller services, in
a microservice architecture. The idea is to have components with a single and well-defined
responsibility, and achieve the same functionality as a monolithic application by making
those services cooperate, and exchange information.

1.2. Abstractions
+++++++++++++++++

This is where encapsulation appears again. From our systems (as we do in relation to the
code), we want to speak in terms of the domain problem, and leave the implementation
details as hidden as possible.

In the same way that the code has to be expressive (almost to the point of being self-
documenting), and have the right abstractions that reveal the solution to the essential
problem (minimizing accidental complexity), the architecture should tell us what the
system is about. Details such as the solution used to persist data on disk, the web
framework of choice, the libraries used to connect to external agents, and interaction
between systems, are not relevant. What is relevant is what the system does. A concept
such as a scream architecture (SCREAM) reflects this idea.

The **dependency inversion principle (DIP)** is of great help in this regard; we don't want to depend upon
concrete implementations but rather abstractions. In the code, we place abstractions (or interfaces)
on the boundaries, the dependencies, those parts of the application that we don't control
and might change in the future. We do this because we want to invert the dependencies.
Let them have to adapt to our code (by having to comply with an interface), and not the
other way round.

Creating abstractions and inverting dependencies are good practices, but they're not
enough. We want our entire application to be independent and isolated from things that are
out of our control. And this is even more than just abstracting with objects—we need layers
of abstraction.

This is a subtle, but important difference with respect to the detailed design. In the DIP, it
was recommended to create an interface, that could be implemented with the abc module
from the standard library, for instance. Because Python works with duck typing, while
using an abstract class might be helpful, it's not mandatory, as we can easily achieve the
same effect with regular objects as long as they comply with the required interface. The
dynamic typing nature of Python allowed us to have these alternatives. When thinking in
terms of architecture, there is no such a thing. As it will become clearer with the example,
we need to abstract dependencies entirely, and there is no feature of Python that can do
that for us.

Some might argue "Well, the ORM is a good abstraction for a database, isn't it?" It's not
enough. The ORM itself is a dependency and, as such, out of our control. It would be even
better to create an intermediate layer, an adapter, between the API of the ORM and our
application.

This means that we don't abstract the database just with an ORM; we use the abstraction
layer we create on top of it, to define objects of our own that belong to our domain.
The application then imports this component, and uses the entities provided by this layer,
but not the other way round. The abstraction layer should not know about the logic of our
application; it's even truer that the database should know nothing about the application
itself. If that were the case, the database would be coupled to our application. The goal is to
invert the dependency—this layer provides an API, and every storage component that
wants to connect has to conform to this API. This is the concept of a hexagonal architecture
(HEX).
