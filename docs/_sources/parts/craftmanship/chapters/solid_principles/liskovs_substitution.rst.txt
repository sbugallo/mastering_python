3. Liskov's substitution principle
**********************************

**Liskov's substitution principle (LSP)** states that there is a series of properties that an object
type must hold to preserve reliability on its design.

The main idea behind LSP is that, for any class, a client should be able to use any of its
subtypes indistinguishably, without even noticing, and therefore without compromising
the expected behavior at runtime. This means that clients are completely isolated and
unaware of changes in the class hierarchy.

More formally, this is the original definition (LISKOV 01) of Liskov's substitution principle::

    if S is a subtype of T, then objects of type T may be replaced by objects of type S, without breaking the program.

This can be understood with the help of a generic diagram such as the following one.

Imagine that there is some client class that requires (includes) objects of another type.
Generally speaking, we will want this client to interact with objects of some type, namely, it
will work through an interface.

Now, this type might as well be just a generic interface definition, an abstract class or an
interface, not a class with the behavior itself. There may be several subclasses extending this
type (described in the diagram with the name Subtype, up to N). The idea behind this
principle is that, if the hierarchy is correctly implemented, the client class has to be able to
work with instances of any of the subclasses without even noticing. These objects should be
interchangeable, as shown here:

.. figure:: ../../../../_static/images/ch4_lsp_diagram.png
   :width: 40%
   :align: center

This is related to other design principles we have already visited, like designing to
interfaces. A good class must define a clear and concise interface, and as long as subclasses
honor that interface, the program will remain correct.

As a consequence of this, the principle also relates to the ideas behind designing by
contract. There is a contract between a given type and a client. By following the rules of
LSP, the design will make sure that subclasses respect the contracts as they are defined by
parent classes.

There are some scenarios so notoriously wrong with respect to the LSP that they can be
easily identified.

3.1. Detecting incorrect datatypes in method signatures
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

By using type annotations throughout our code, we can quickly detect some basic errors early and check basic
compliance with LSP.

One common code smell is that one of the subclasses of the parent class were to override a method in an incompatible
fashion:

.. code-block:: python

    class Event:
        ...
        def meets_condition(self, event_data: dict) -> bool:
            return False

    class LoginEvent(Event):

        def meets_condition(self, event_data: list) -> bool:
            return bool(event_data)

The violation to LSP is clear—since the derived class is using a type for the ``event_data``
parameter which is different from the one defined on the base class, we cannot expect them
to work equally. Remember that, according to this principle, any caller of this hierarchy has
to be able to work with ``Event`` or ``LoginEvent`` transparently, without noticing any
difference. Interchanging objects of these two types should not make the application fail.
Failure to do so would break the polymorphism on the hierarchy.

The same error would have occurred if the return type was changed for something other
than a Boolean value. The rationale is that clients of this code are expecting a Boolean value
to work with. If one of the derived classes changes this return type, it would be breaking
the contract, and again, we cannot expect the program to continue working normally.

A quick note about types that are not the same but share a common interface: even though
this is just a simple example to demonstrate the error, it is still true that both dictionaries
and lists have something in common; they are both iterables. This means that in some cases,
it might be valid to have a method that expects a dictionary and another one expecting to
receive a list, as long as both treat the parameters through the iterable interface. In this case,
the problem would not lie in the logic itself (LSP might still apply), but in the definition of
the types of the signature, which should read neither ``list`` nor ``dict``, but a union of both.
Regardless of the case, something has to be modified, whether it is the code of the method,
the entire design, or just the type annotations.

Another strong violation of LSP is when, instead of varying the types of the parameters on
the hierarchy, the signatures of the methods differ completely. This might seem like quite a
blunder, but detecting it would not always be so easy to remember; Python is interpreted,
so there is no compiler to detect these type of error early on, and therefore they will not be
caught until runtime.

In the presence of a class that breaks the compatibility defined by the hierarchy (for
example, by changing the signature of the method, adding an extra parameter, and so on)
shown as follows:

.. code-block:: python

    class LogoutEvent(Event):
        def meets_condition(self, event_data: dict, override: bool) -> bool:
            if override:
                return True

3.2. More subtle cases of LSP violations
++++++++++++++++++++++++++++++++++++++++

Cases where contracts are modified are particularly harder to detect. Given
that the entire idea of LSP is that subclasses can be used by clients just like their parent
class, it must also be true that contracts are correctly preserved on the hierarchy.

Remember that, when designing by contract,
the contract between the client and supplier sets some rules: the client must provide the
preconditions to the method, which the supplier might validate, and it returns some result
to the client that it will check in the form of postconditions.

The parent class defines a contract with its clients. Subclasses of this one must respect such
a contract. This means that, for example:

- A subclass can never make preconditions stricter than they are defined on the parent class
- A subclass can never make postconditions weaker than they are defined on the parent class

Consider the example of the events hierarchy defined in the previous section, but now with
a change to illustrate the relationship between LSP and DbC.

This time, we are going to assume a precondition for the method that checks the criteria
based on the data, that the provided parameter must be a dictionary that contains both keys
"before" and "after", and that their values are also nested dictionaries. This allows us to
encapsulate even further, because now the client does not need to catch the ``KeyError``
exception, but instead just calls the precondition method (assuming that is acceptable to fail
if the system is operating under the wrong assumptions). As a side note, it is good that we
can remove this from the client, as now, ``SystemMonitor`` does not require to know which
types of exceptions the methods of the collaborator class might raise (remember that
exception weaken encapsulation, as they require the caller to know something extra about
the object they are calling).

Such a design might be represented with the following changes in the code:

.. code-block:: python

    class Event:

        def __init__(self, raw_data):
            self.raw_data = raw_data

        @staticmethod
        def meets_condition(event_data: dict):
            return False

        @staticmethod
        def meets_condition_pre(event_data: dict):
            """Precondition of the contract of this interface.
            Validate that the ``event_data`` parameter is properly formed.
            """
            assert isinstance(event_data, dict), f"{event_data!r} is not a dict"
            for moment in ("before", "after"):
                assert moment in event_data, f"{moment} not in {event_data}"
                assert isinstance(event_data[moment], dict)

And now the code that tries to detect the correct event type just checks the precondition
once, and proceeds to find the right type of event:

.. code-block:: python

    class SystemMonitor:
        """Identify events that occurred in the system."""
        def __init__(self, event_data):
            self.event_data = event_data

        def identify_event(self):
            Event.meets_condition_pre(self.event_data)
            event_cls = next((event_cls for event_cls in Event.__subclasses__()
                if event_cls.meets_condition(self.event_data)), UnknownEvent)

            return event_cls(self.event_data)

The contract only states that the top-level keys "before" and "after" are mandatory and
that their values should also be dictionaries. Any attempt in the subclasses to demand a
more restrictive parameter will fail.

The class for the transaction event was originally correctly designed. Look at how the code
does not impose a restriction on the internal key named "transaction"; it only uses its
value if it is there, but this is not mandatory:

.. code-block:: python

    class TransactionEvent(Event):
        """Represents a transaction that has just occurred on the system."""

        @staticmethod
        def meets_condition(event_data: dict):
            return event_data["after"].get("transaction") is not None

However, the original two methods are not correct, because they demand the presence of a
key named "session", which is not part of the original contract. This breaks the contract,
and now the client cannot use these classes in the same way it uses the rest of them because
it will raise ``KeyError``.

After fixing this (changing the square brackets for the .get() method), the order on the
LSP has been reestablished, and polymorphism prevails:

.. code-block:: python

    >>> l1 = SystemMonitor({"before": {"session": 0}, "after": {"session": 1}})
    >>> l1.identify_event().__class__.__name__
    'LoginEvent'
    >>> l2 = SystemMonitor({"before": {"session": 1}, "after": {"session": 0}})
    >>> l2.identify_event().__class__.__name__
    'LogoutEvent'
    >>> l3 = SystemMonitor({"before": {"session": 1}, "after": {"session": 1}})
    >>> l3.identify_event().__class__.__name__
    'UnknownEvent'
    >>> l4 = SystemMonitor({"before": {}, "after": {"transaction": "Tx001"}})
    >>> l4.identify_event().__class__.__name__
    'TransactionEvent'

We have to be careful when designing classes that we do
not accidentally change the input or output of the methods in a way that would be
incompatible with what the clients are originally expecting.

3.3. Remarks on the LSP
+++++++++++++++++++++++

The LSP is fundamental to a good object-oriented software design because it emphasizes
one of its core traits—polymorphism. It is about creating correct hierarchies so that classes
derived from a base one are polymorphic along the parent one, with respect to the methods
on their interface.

It is also interesting to notice how this principle relates to the previous one—if we attempt
to extend a class with a new one that is incompatible, it will fail, the contract with the client
will be broken, and as a result such an extension will not be possible (or, to make it
possible, we would have to break the other end of the principle and modify code in the
client that should be closed for modification, which is completely undesirable and
unacceptable).

Carefully thinking about new classes in the way that LSP suggests helps us to extend the
hierarchy correctly. We could then say that LSP contributes to the OCP.