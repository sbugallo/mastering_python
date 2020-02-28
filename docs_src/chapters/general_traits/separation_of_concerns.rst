3. Separation of concerns
*************************

This is a design principle that is applied at multiple levels. It is not just about the low-level design
(code), but it is also relevant at a higher level of abstraction, so it will come up later when we talk about
architecture.

Different responsibilities should go into different components, layers, or modules of the application. Each
part of the program should only be responsible for a part of the functionality (what we call its concerns) and
should know nothing about the rest.

The goal of separating concerns in software is to enhance maintainability by minimizing ripple effects. A
ripple effect means the propagation of a change in the software from a starting point. This could be the case
of an error or exception triggering a chain of other exceptions, causing failures that will result in a defect
on a remote part of the application. It can also be that we have to change a lot of code scattered through
multiple parts of the code base, as a result of a simple change in a function definition.

Clearly, we do not want these scenarios to happen. The software has to be easy to change. If we have to modify
or refactor some part of the code that has to have a minimal impact on the rest of the application, the way to
achieve this is through proper encapsulation.

In a similar way, we want any potential errors to be contained so that they don't cause major damage.

This concept is related to the DbC principle in the sense that each concern can be enforced by a contract.
When a contract is violated, and an exception is raised as a result of such a violation, we know what part of
the program has the failure, and what responsibilities failed to be met.

Despite this similarity, separation of concerns goes further. We normally think of contracts between
functions, methods, or classes, and while this also applies to responsibilities that have to be separated, the
idea of separation of concerns also applies to Python modules, packages, and basically any software component.

3.1. Cohesion and coupling
++++++++++++++++++++++++++

These are important concepts for good software design.

On the one hand, cohesion means that objects should have a small and well-defined purpose, and they should do
as little as possible. It follows a similar philosophy as Unix commands that do only one thing and do it well.
The more cohesive our objects are, the more useful and reusable they become, making our design better.

On the other hand, coupling refers to the idea of how two or more objects depend on each other. This
dependency poses a limitation. If two parts of the code (objects or methods) are too dependent on each other,
they bring with them some undesired consequences:

- **No code reuse**: If one function depends too much on a particular object, or takes too many parameters, it's coupled with this object, which means that it will be really difficult to use that function in a different context (in order to do so, we will have to find a suitable parameter that complies with a very restrictive interface).
- **Ripple effects**: Changes in one of the two parts will certainly impact the other, as they are too close
- **Low level of abstraction**: When two functions are so closely related, it is hard to see them as different concerns resolving problems at different levels of abstraction

.. note:: Rule of thumb: Well-defined software will achieve high cohesion and low coupling.