5. Composition and inheritance
******************************

In object-oriented software design, there are often discussions as to how to address some problems by using
the main ideas of the paradigm (polymorphism, inheritance, and encapsulation).

Probably the most commonly used of these ideas is inheritance—developers often start by creating a class
hierarchy with the classes they are going to need and decide the methods each one should implement.

While inheritance is a powerful concept, it does come with its perils. The main one is that every time we
extend a base class, we are creating a new one that is tightly coupled with the parent. As we have already
discussed, coupling is one of the things we want to reduce to a minimum when designing software.

One of the main uses developers relate inheritance with is code reuse. While we should always embrace code
reuse, it is not a good idea to force our design to use inheritance to reuse code just because we get the
methods from the parent class for free. The proper way to reuse code is to have highly cohesive objects that
can be easily composed and that could work on multiple contexts.

5.1. When inheritance is a good decision
+++++++++++++++++++++++++++++++++++++++++

We have to be careful when creating a derived class, because this is a double-edged sword—on the one hand, it
has the advantage that we get all the code of the methods from the parent class for free, but on the other
hand, we are carrying all of them to a new class, meaning that we might be placing too much functionality in a
new definition.

When creating a new subclass, we have to think if it is actually going to use all of the methods it has just
inherited, as a heuristic to see if the class is correctly defined. If instead, we find out that we do not
need most of the methods, and have to override or replace them, this is a design mistake that could be caused
by several reasons:

- The superclass is vaguely defined and contains too much responsibility, instead of a well-defined interface.
- The subclass is not a proper specialization of the superclass it is trying to extend.

A good case for using inheritance is the type of situation when you have a class that defines certain
components with its behavior that are defined by the interface of this class (its public methods and
attributes), and then you need to specialize this class in order to create objects that do the same but with
something else added, or with some particular parts of its behavior changed.

You can find examples of good uses of inheritance in the Python standard library itself. For example, in the
``http.server`` package, we can find a base class such as ``BaseHTTPRequestHandler``, and subclasses such as
``SimpleHTTPRequestHandler`` that extend this one by adding or changing part of its base interface.

Speaking of interface definition, this is another good use for inheritance. When we want to enforce the
interface of some objects, we can create an abstract base class that does not implement the behavior itself,
but instead just defines the interface—every class that extends this one will have to implement these to be a
proper subtype.

Finally, another good case for inheritance is exceptions. We can see that the standard exception in Python
derives from ``Exception``. This is what allows you to have a generic clause such as ``except Exception:``,
which will catch every possible error. The important point is the conceptual one, they are classes derived
from Exception because they are more specific exceptions. This also works in well-known libraries such as
``requests``, for instance, in which an ``HTTPError`` is ``RequestException``, which in turn is an ``IOError``.

5.2. Anti-patterns for inheritance
+++++++++++++++++++++++++++++++++++

If the previous section had to be summarized into a single word, it would be specialization. The correct use
for inheritance is to specialize objects and create more detailed abstractions starting from base ones.

The parent (or base) class is part of the public definition of the new derived class. This is because the
methods that are inherited will be part of the interface of this new class. For this reason, when we read the
public methods of a class, they have to be consistent with what the parent class defines.

For example, if we see that a class derived from ``BaseHTTPRequestHandler`` implements a method named
``handle()``, it would make sense because it is overriding one of the parents. If it had any other method
whose name relates to an action that has to do with an HTTP request, then we could also think that is
correctly placed (but we would not think that if we found something called process_purchase() on that class).

The previous illustration might seem obvious, but it is something that happens very often, especially when
developers try to use inheritance with the sole goal of reusing code. In the next example, we will see a
typical situation that represents a common anti-pattern in Python: there is a domain problem that has to be
represented, and a suitable data structure is devised for that problem, but instead of creating an object that
uses such a data structure, the object becomes the data structure itself.

Let's see these problems more concretely through an example. Imagine we have a system for managing insurance,
with a module in charge of applying policies to different clients. We need to keep in memory a set of
customers that are being processed at the time in order to apply those changes before further processing or
persistence. The basic operations we need are to store a new customer with its records as satellite data,
apply a change on a policy, or edit some of the data, just to name a few. We also need to support a batch
operation, that is, when something on the policy itself changes (the one this module is currently processing),
we have to apply these changes overall to customers on the current transaction.

Thinking in terms of the data structure we need, we realize that accessing the record for a particular
customer in constant time is a nice trait. Therefore, something like ``policy_transaction[customer_id]``
looks like a nice interface. From this, we might think that a subscriptable object is a good idea, and further
on, we might get carried away into thinking that the object we need is a dictionary:

.. code-block:: python

    class TransactionalPolicy(collections.UserDict):
        """Example of an incorrect use of inheritance."""

        def change_in_policy(self, customer_id, **new_policy_data):
            self[customer_id].update(**new_policy_data)

With this code, we can get information about a policy for a customer by its identifier:

.. code-block:: python

    >>> policy = TransactionalPolicy({
    ...
    "client001": {
    ...
    "fee": 1000.0,
    ...
    "expiration_date": datetime(2020, 1, 3),
    ...
    }
    ... })

    >>> policy["client001"]
    {'fee': 1000.0, 'expiration_date': datetime.datetime(2020, 1, 3, 0, 0)}

    >>> policy.change_in_policy("client001", expiration_date=datetime(2020, 1,
    4))

    >>> policy["client001"]
    {'fee': 1000.0, 'expiration_date': datetime.datetime(2020, 1, 4, 0, 0)}

Sure, we achieved the interface we wanted in the first place, but at what cost? Now, this class has a lot of
extra behavior from carrying out methods that weren't necessary:

.. code-block:: python

    >>> dir(policy)
    [ # all magic and special method have been omitted for brevity...
    'change_in_policy', 'clear', 'copy', 'data', 'fromkeys', 'get', 'items',
    'keys', 'pop', 'popitem', 'setdefault', 'update', 'values']

There are (at least) two major problems with this design. On the one hand, the hierarchy is wrong. Creating a
new class from a base one conceptually means that it's a more specific version of the class it's extending
(hence the name). How is it that a ``TransactionalPolicy`` is a dictionary? Does this make sense? Remember,
this is part of the public interface of the object, so users will see this class, their hierarchy, and will
notice such an odd specialization, as well as its public methods.

This leads us to the second problem—coupling. The interface of the transactional policy now includes all
methods from a dictionary. Does a transactional policy really need methods such as ``pop()`` or ``items()``?
However, there they are. They are also public, so any user of this interface is entitled to call them, with
whatever undesired side-effect they may carry. More on this point: we don't really gain much by extending a
dictionary. The only method it actually needs to update for all customers affected by a change in the current
policy (``change_in_policy()``) is not on the base class, so we will have to define it ourselves either way.

This is a problem of mixing implementation objects with domain objects. A dictionary is an implementation
object, a data structure, suitable for certain kinds of operation, and with a trade-off like all data
structures. A transactional policy should represent something in the domain problem, an entity that is part of
the problem we are trying to solve.

Hierarchies like this one are incorrect, and just because we get a few magic methods from a base class (to
make the object subscriptable by extending a dictionary) is not reason enough to create such an extension.
Implementation classes should be extending solely when creating other, more specific, implementation classes.
In other words, extend a dictionary if you want to create another (more specific, or slightly modified)
dictionary. The same rule applies to classes of the domain problem.

The correct solution here is to use composition. ``TransactionalPolicy`` is not a dictionary: it uses a
dictionary. It should store a dictionary in a private attribute, and implement __getitem__() by proxying from
that dictionary and then only implementing the rest of the public method it requires:

.. code-block:: python

    class TransactionalPolicy:
        """Example refactored to use composition."""

        def __init__(self, policy_data, **extra_data):
            self._data = {**policy_data, **extra_data}

        def change_in_policy(self, customer_id, **new_policy_data):
            self._data[customer_id].update(**new_policy_data)

        def __getitem__(self, customer_id):
            return self._data[customer_id]

        def __len__(self):
            return len(self._data)

This way is not only conceptually correct, but also more extensible. If the underlying data structure (which,
for now, is a dictionary) is changed in the future, callers of this object will not be affected, so long as
the interface is maintained. This reduces coupling, minimizes ripple effects, allows for better refactoring
(unit tests ought not to be changed), and makes the code more maintainable.

5.3. Multiple inheritance in Python
+++++++++++++++++++++++++++++++++++

Python supports multiple inheritance. As inheritance, when improperly used, leads to design problems, you
could also expect that multiple inheritance will also yield even bigger problems when it's not correctly
implemented.

Multiple inheritance is, therefore, a double-edged sword. It can also be very beneficial in some cases. Just
to be clear, there is nothing wrong with multiple inheritance, the only problem it has is that when it's not
implemented correctly, it will multiply the problems.

Multiple inheritance is a perfectly valid solution when used correctly, and this opens up new patterns
(such as the adapter pattern) and mixins.

One of the most powerful applications of multiple inheritance is perhaps that which enables the creation of
mixins. Before exploring mixins, we need to understand how multiple inheritance works, and how methods are
resolved in a complex hierarchy.

5.3.1. Method Resolution Order (MRO)
------------------------------------

Some people don't like multiple inheritance because of the constraints it has in other programming languages,
for instance, the so-called diamond problem. When a class extends from two or more, and all of those classes
also extend from other base classes, the bottom ones will have multiple ways to resolve the methods coming
from the top-level classes. The question is, which of these implementations is used?

Consider the following diagram, which has a structure with multiple inheritance.

.. figure:: ../../_static/images/ch3_diagram.png
   :width: 50%
   :align: center

The top-level class has a class attribute and implements the ``__str__`` method. Think of any of the concrete
classes, for example, ``ConcreteModuleA12``: it extends from ``BaseModule1`` and ``BaseModule2``, and each one
of them will take the implementation of ``__str__`` from ``BaseModule``. Which of these two methods is going
to be the one for ``ConcreteModuleA12``?

With the value of the class attribute, this will become evident:

.. code-block:: python

    class BaseModule:
        module_name = "top"

        def __init__(self, module_name):
            self.name = module_name

        def __str__(self):
            return f"{self.module_name}:{self.name}"

    class BaseModule1(BaseModule):
        module_name = "module-1"

    class BaseModule2(BaseModule):
        module_name = "module-2"

    class BaseModule3(BaseModule):
        module_name = "module-3"

    class ConcreteModuleA12(BaseModule1, BaseModule2):
        """Extend 1 & 2"""

    class ConcreteModuleB23(BaseModule2, BaseModule3):
        """Extend 2 & 3"""


Now, let's test this to see what method is being called:

.. code-block:: python

    >>> str(ConcreteModuleA12("test"))
    'module-1:test'

There is no collision. Python resolves this by using an algorithm called C3 linearization or MRO, which
defines a deterministic way in which methods are going to be called.

In fact, we can specifically ask the class for its resolution order:

.. code-block:: python

    >>> [cls.__name__ for cls in ConcreteModuleA12.mro()]
    ['ConcreteModuleA12', 'BaseModule1', 'BaseModule2', 'BaseModule', 'object']

Knowing about how the method is going to be resolved in a hierarchy can be used to our advantage when
designing classes because we can make use of mixins.

5.3.2. Mixins
-------------

A mixin is a base class that encapsulates some common behavior with the goal of reusing code. Typically, a
mixin class is not useful on its own, and extending this class alone will certainly not work, because most of
the time it depends on methods and properties that are defined in other classes. The idea is to use mixin
classes along with other ones, through multiple inheritance, so that the methods or properties used on the
mixin will be available.

Imagine we have a simple parser that takes a string and provides iteration over it by its values separated by
hyphens (-):

.. code-block:: python

    class BaseTokenizer:
        def __init__(self, str_token):
            self.str_token = str_token
        def __iter__(self):
        yield from self.str_token.split("-")

This is quite straightforward:

.. code-block:: python

    >>> tk = BaseTokenizer("28a2320b-fd3f-4627-9792-a2b38e3c46b0")
    >>> list(tk)
    ['28a2320b', 'fd3f', '4627', '9792', 'a2b38e3c46b0']

But now we want the values to be sent in upper-case, without altering the base class. For this simple example,
we could just create a new class, but imagine that a lot of classes are already extending from
``BaseTokenizer``, and we don't want to replace all of them. We can mix a new class into the hierarchy that
handles this transformation:

.. code-block:: python

    class UpperIterableMixin:
        def __iter__(self):
            return map(str.upper, super().__iter__())

    class Tokenizer(UpperIterableMixin, BaseTokenizer):
        pass

The new ``Tokenizer`` class is really simple. It doesn't need any code because it takes advantage of the mixin.
This type of mixing acts as a sort of decorator. Based on what we just saw, ``Tokenizer`` will take ``__iter__``
from the mixin, and this one, in turn, delegates to the next class on the line (by calling ``super()``), which
is the ``BaseTokenizer``, but it converts its values to uppercase, creating the desired effect.
