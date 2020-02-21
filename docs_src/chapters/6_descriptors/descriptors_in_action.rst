3. Descriptors in action
************************

Now that we have seen what descriptors are, how they work, and what the main ideas
behind them are, we can see them in action. In this section, we will be exploring some
situations that can be elegantly addressed through descriptors.

Here, we will look at some examples of working with descriptors, and we will also cover
implementation considerations for them (different ways of creating them, with their pros
and cons), and finally we will discuss what are the most suitable scenarios for descriptors.

3.1. An application of descriptors
++++++++++++++++++++++++++++++++++

We will start with a simple example that works, but that will lead to some code duplication.
It is not very clear how this issue will be addressed. Later on, we will devise a way of
abstracting the repeated logic into a descriptor, which will address the duplication
problem, and we will notice that the code on our client classes will be reduced drastically.

3.1.1. A first attempt without using descriptors
------------------------------------------------

The problem we want to solve now is that we have a regular class with some attributes, but
we wish to track all of the different values a particular attribute has over time, for example,
in a list. The first solution that comes to our mind is to use a property, and every time a
value is changed for that attribute in the setter method of the property, we add it to an
internal list that will keep this trace as we want it.

Imagine that our class represents a traveler in our application that has a current city, and
we want to keep track of all the cities that user has visited throughout the running of the
program. The following code is a possible implementation that addresses these
requirements:

.. code-block:: python

    class Traveller:
        def __init__(self, name, current_city):
            self.name = name
            self._current_city = current_city
            self._cities_visited = [current_city]

        @property
        def current_city(self):
            return self._current_city

        @current_city.setter
        def current_city(self, new_city):
            if new_city != self._current_city:
                self._cities_visited.append(new_city)
            self._current_city = new_city

        @property
        def cities_visited(self):
            return self._cities_visited

We can easily check that this code works according to our requirements:

.. code-block:: python

    >>> alice = Traveller("Alice", "Barcelona")
    >>> alice.current_city = "Paris"
    >>> alice.current_city = "Brussels"
    >>> alice.current_city = "Amsterdam"
    >>> alice.cities_visited
    ['Barcelona', 'Paris', 'Brussels', 'Amsterdam']

So far, this is all we need and nothing else has to be implemented. For the purposes of this
problem, the property would be more than enough. What happens if we need the exact
same logic in multiple places of the application? This would mean that this is actually an
instance of a more generic problem: tracing all the values of an attribute in another one.
What would happen if we want to do the same with other attributes, such as keeping track
of all tickets Alice bought, or all the countries she has been in? We would have to repeat the
logic in all of these places.

Moreover, what would happen if we need this same behavior in different classes? We
would have to repeat the code or come up with a generic solution (maybe a decorator, a
property builder, or a descriptor).

3.1.2. The idiomatic implementation
-----------------------------------

We will now look at how to address the questions of the previous section by using a
descriptor that is generic enough as to be applied in any class. Again, this example is not
really needed because the requirements do not specify such generic behavior (we haven't
even followed the rule of three instances of the similar pattern previously creating the
abstraction), but it is shown with the goal of portraying descriptors in action.

.. note:: Do not implement a descriptor unless there is actual evidence of the repetition we are trying to solve, and the complexity is proven to have paid off.

Now, we will create a generic descriptor that, given a name for the attribute to hold the
traces of another one, will store the different values of the attribute in a list.

As we mentioned previously, the code is more than what we need for the problem, but its
intention is just to show how a descriptor would help us in this case. Given the generic
nature of descriptors, the reader will notice that the logic on it (the name of their method,
and attributes) does not relate to the domain problem at hand (a traveler object). This is
because the idea of the descriptor is to be able to use it in any type of class, probably on
different projects, with the same outcomes.

In order to address this gap, some parts of the code are annotated, and the respective
explanation for each section (what it does, and how it relates to the original problem) is
described in the following code:

.. code-block:: python

    class HistoryTracedAttribute:
        def __init__(self, trace_attribute_name) -> None:
            self.trace_attribute_name = trace_attribute_name # [1]
            self._name = None

        def __set_name__(self, owner, name):
            self._name = name

        def __get__(self, instance, owner):
            if instance is None:
                return self

            return instance.__dict__[self._name]

        def __set__(self, instance, value):
            self._track_change_in_value_for_instance(instance, value)
            instance.__dict__[self._name] = value

        def _track_change_in_value_for_instance(self, instance, value):
            self._set_default(instance) # [2]
            if self._needs_to_track_change(instance, value):
                instance.__dict__[self.trace_attribute_name].append(value)

        def _needs_to_track_change(self, instance, value) -> bool:
            try:
                current_value = instance.__dict__[self._name]
            except KeyError: # [3]
                return True

            return value != current_value # [4]

        def _set_default(self, instance):
            instance.__dict__.setdefault(self.trace_attribute_name, []) # [6]

    class Traveller:
        current_city = HistoryTracedAttribute("cities_visited") # [1]

        def __init__(self, name, current_city):
            self.name = name
            self.current_city = current_city # [5]

Some annotations and comments on the code are as follows (numbers in the list correspond
to the number annotations in the previous listing):

1. The name of the attribute is one of the variables assigned to the descriptor, in this case, ``current_city``. We pass to the descriptor the name of the variable in which it will store the trace for the variable of the descriptor. In this example, we are telling our object to keep track of all the values that ``current_city`` has had in the attribute named ``cities_visited``.
2. The first time we call the descriptor, in the ``init``, the attribute for tracing values will not exist, in which case we initialize it to an empty list to later append values to it.
3. In the ``init`` method, the name of the attribute ``current_city`` will not exist either, so we want to keep track of this change as well. This is the equivalent of initializing the list with the first value in the previous example.
4. Only track changes when the new value is different from the one that is currently set.
5. In the ``init method``, the descriptor already exists, and this assignment instruction triggers the actions from step 2 (create the empty list to start tracking values for it), and step 3 (append the value to this list, and set it to the key in the object for retrieval later).
6. The ``setdefault`` method in a dictionary is used to avoid a ``KeyError``. In this case an empty list will be returned for those attributes that aren't still available.

It is true that the code in the descriptor is rather complex. On the other hand, the code in
the client class is considerably simpler. Of course, this balance only pays off if we are
going to use this descriptor multiple times, which is a concern we have already covered.

What might not be so clear at this point is that the descriptor is indeed completely
independent from the client class. Nothing in it suggests anything about the business
logic. This makes it perfectly suitable to apply it in any other class; even if it does
something completely different, the descriptor will take the same effect.

This is the true Pythonic nature of descriptors. They are more appropriate for defining
libraries, frameworks, or internal APIs, and not that much for business logic.

3.2. Different forms of implementing descriptors
++++++++++++++++++++++++++++++++++++++++++++++++

We have to first understand a common issue that's specific to the nature of descriptors
before thinking of ways of implementing them. First, we will discuss the problem of a
global shared state, and afterward we will move on and look at different ways descriptors
can be implemented while taking this into consideration.

3.2.1. The issue of global shared state
+++++++++++++++++++++++++++++++++++++++

As we have already mentioned, descriptors need to be set as class attributes to work. This
should not be a problem most of the time, but it does come with some warnings that need
to be taken into consideration.

The problem with class attributes is that they are shared across all instances of that class.
Descriptors are not an exception here, so if we try to keep data in a descriptor object,
keep in mind that all of them will have access to the same value.

Let's see what happens when we incorrectly define a descriptor that keeps the data itself,
instead of storing it in each object:

.. code-block::

    class SharedDataDescriptor:
        def __init__(self, initial_value):
            self.value = initial_value

        def __get__(self, instance, owner):
            if instance is None:
                return self
            return self.value

        def __set__(self, instance, value):
            self.value = value

        class ClientClass:
            descriptor = SharedDataDescriptor("first value")

In this example, the descriptor object stores the data itself. This carries with it the
inconvenience that when we modify the value for an instance all other instances of the
same classes are also modified with this value as well. The following code listing puts that
theory in action:

.. code-block:: python

    >>> client1 = ClientClass()
    >>> client1.descriptor
    'first value'
    >>> client2 = ClientClass()
    >>> client2.descriptor
    'first value'
    >>> client2.descriptor = "value for client 2"
    >>> client2.descriptor
    'value for client 2'
    >>> client1.descriptor
    'value for client 2'

Notice how we change one object, and suddenly all of them are from the same class, and
we can see that this value is reflected. This is because ``ClientClass.descriptor`` is
unique; it's the same object for all of them.

In some cases, this might be what we actually want (for instance, if we were to create a sort
of Borg pattern implementation, on which we want to share state across all objects from a
class), but in general, that is not the case, and we need to differentiate between objects.

To achieve this, the descriptor needs to know the value for each instance and return it
accordingly. That is the reason we have been operating with the dictionary (``__dict__``) of
each instance and setting and retrieving the values from there.

This is the most common approach. We have already covered why we cannot use
``getattr()`` and ``setattr()`` on those methods, so modifying the ``__dict__`` attribute is the
last standing option, and, in this case, is acceptable.

3.2.2. Accessing the dictionary of the object
---------------------------------------------

The way we implement descriptors is making the descriptor object
store the values in the dictionary of the object, ``__dict__``, and retrieve the parameters from
there as well.

.. note:: Always store and return the data from the ``__dict__`` attribute of the instance.

3.2.3. Using weak references
----------------------------

Another alternative (if we don't want to use ``__dict__``) is to make the descriptor object
keep track of the values for each instance itself, in an internal mapping, and return values
from this mapping as well.

There is a caveat, though. This mapping cannot just be any dictionary. Since the client
class has a reference to the descriptor, and now the descriptor will keep references to the
objects that use it, this will create circular dependencies, and, as a result, these objects will
never be garbage-collected because they are pointing at each other.

In order to address this, the dictionary has to be a weak key one, as defined in the
``weakref`` module.

In this case, the code for the descriptor might look like the following:

.. code-block:: python

    from weakref import WeakKeyDictionary

    class DescriptorClass:
        def __init__(self, initial_value):
            self.value = initial_value
            self.mapping = WeakKeyDictionary()

        def __get__(self, instance, owner):
            if instance is None:
                return self
            return self.mapping.get(instance, self.value)

        def __set__(self, instance, value):
            self.mapping[instance] = value

This addresses the issues, but it does come with some considerations:

- The objects no longer hold their attributes: the descriptor does instead. This is somewhat controversial, and it might not be entirely accurate from a conceptual point of view. If we forget this detail, we might be asking the object by inspecting its dictionary, trying to find things that just aren't there (calling ``vars(client)`` will not return the complete data, for example).
- It poses the requirement over the objects that they need to be hashable. If they aren't, they can't be part of the mapping. This might be too demanding a requirement for some applications.

For these reasons, we prefer the implementation that has been shown so far,
which uses the dictionary of each instance. However, for completeness, we have shown this
alternative as well.

3.3. More considerations about descriptors
++++++++++++++++++++++++++++++++++++++++++

Here, we will discuss general considerations about descriptors in terms of what we can do
with them when it is a good idea to use them, and also how things that we might have
initially conceived as having been resolved by means of another approach can be improved
through descriptors. We will then analyze the pros and cons of the original implementation
versus the one after descriptors have been used.

3.3.1. Reusing code
-------------------

Descriptors are a generic tool and a powerful abstraction that we can use to avoid code
duplication. The best way to decide when to use descriptors is to identify cases where we
would be using a property (whether for its get logic, set logic, or both), but repeating its
structure many times.

Properties are just a particular case of descriptors (the @property decorator is a descriptor
that implements the full descriptor protocol to define their get, set, and delete actions),
which means that we can use descriptors for far more complex tasks.

Another powerful type we have seen for reusing code was decorators. Descriptors can help us create to better
decorators by making sure that they will be able to work correctly for class methods as
well.

When it comes to decorators, we could say that it is safe to always implement the
``__get__()`` method on them, and also make it a descriptor. When trying to decide whether
the decorator is worth creating, consider the problems but note that there are no extra considerations
toward descriptors.

As for generic descriptors, besides the aforementioned three instances rule that applies to
decorators (and, in general, any reusable component), it is advisable to also keep in mind
that you should use descriptors for cases when we want to define an internal API, which is
some code that will have clients consuming it. This is a feature-oriented more toward
designing libraries and frameworks, rather than one-time solutions.

Unless there is a very good reason to, or that the code will look significantly better, we
should avoid putting business logic in a descriptor. Instead, the code of a descriptor will
contain more implementational code rather than business code. It is more similar to
defining a new data structure or object that another part of our business logic will use as a
tool.

.. note:: In general, descriptors will contain implementation logic, and not so much business logic.

3.3.2. Avoiding class decorators
--------------------------------

If we recall the class decorator we used previously to determine how an event object is going to be
serialized, we ended up with an implementation that (for Python 3.7+) relied on two class decorators:

.. code-block:: python

    @Serialization(
        username=show_original,
        password=hide_field,
        ip=show_original,
        timestamp=format_time
    )
    @dataclass
    class LoginEvent:
        username: str
        password: str
        ip: str
        timestamp: datetime

The first one takes the attributes from the annotations to declare the variables, whereas the
second one defines how to treat each file. Let's see whether we can change these two
decorators for descriptors instead.

The idea is to create a descriptor that will apply the transformation over the values of each
attribute, returning the modified version according to our requirements (for example,
hiding sensitive information, and formatting dates correctly):

.. code-block:: python

    from functools import partial
    from typing import Callable


    class BaseFieldTransformation:
        def __init__(self, transformation: Callable[[], str]) -> None:
            self._name = None
            self.transformation = transformation

        def __get__(self, instance, owner):
            if instance is None:
                return self

            raw_value = instance.__dict__[self._name]
            return self.transformation(raw_value)

        def __set_name__(self, owner, name):
            self._name = name

        def __set__(self, instance, value):
            instance.__dict__[self._name] = value
            ShowOriginal = partial(BaseFieldTransformation, transformation=lambda x: x)
            HideField = partial(
                BaseFieldTransformation, transformation=lambda x: "**redacted**"
            )
            FormatTime = partial(
                BaseFieldTransformation,
                transformation=lambda ft: ft.strftime("%Y-%m-%d %H:%M"),
            )

This descriptor is interesting. It was created with a function that takes one argument and
returns one value. This function will be the transformation we want to apply to the field.
From the base definition that defines generically how it is going to work, the rest of the
descriptor classes are defined, simply by changing the particular function each one
needs.

The example uses ``functools.partial`` as a way of simulating sub-classes, by applying a
partial application of the transformation function for that class, leaving a new callable that
can be instantiated directly.

In order to keep the example simple, we will implement the ``__init__()`` and
``serialize()`` methods, although they could be abstracted away as well. Under these
considerations, the class for the event will now be defined as follows:

.. code-block:: python

    class LoginEvent:

        username = ShowOriginal()
        password = HideField()
        ip = ShowOriginal()
        timestamp = FormatTime()

        def __init__(self, username, password, ip, timestamp):
            self.username = username
            self.password = password
            self.ip = ip
            self.timestamp = timestamp

        def serialize(self):
            return {
                "username": self.username,
                "password": self.password,
                "ip": self.ip,
                "timestamp": self.timestamp,
            }

We can see how the object behaves at runtime:

.. code-block:: python

    >>> le = LoginEvent("john", "secret password", "1.1.1.1",
    datetime.utcnow())
    >>> vars(le)
    {'username': 'john', 'password': 'secret password', 'ip': '1.1.1.1',
    'timestamp': ...}
    >>> le.serialize()
    {'username': 'john', 'password': '**redacted**', 'ip': '1.1.1.1',
    'timestamp': '...'}
    >>> le.password
    '**redacted**'

There are some differences with respect to the previous implementation that used a
decorator. This example added the ``serialize()`` method and hid the fields before
presenting them to its resulting dictionary, but if we asked for any of these attributes to an
instance of the event in memory at any point, it would still give us the original value,
without any transformation applied to it (we could have chosen to apply the
transformation when setting the value, and return it directly on the ``__get__()``, as well).

Depending on the sensitivity of the application, this may or may not be acceptable, but in
this case, when we ask the object for its public attributes, the descriptor will apply the
transformation before presenting the results. It is still possible to access the original values
by asking for the dictionary of the object (by accessing ``__dict__``), but when we ask for the
value, by default, it will return it converted.

In this example, all descriptors follow a common logic, which is defined in the base class.
The descriptor should store the value in the object and then ask for it, applying the
transformation it defines. We could create a hierarchy of classes, each one defining its own
conversion function, in a way that the template method design pattern works. In this case,
since the changes in the derived classes are relatively small (just one function), we opted for
creating the derived classes as partial applications of the base class. Creating any new
transformation field should be as simple as defining a new class that will be the base class,
which is partially applied with the function we need. This can even be done ad hoc, so there
might be no need to set a name for it.

Regardless of this implementation, the point is that since descriptors are objects, we can
create models, and apply all rules of object-oriented programming to them. Design patterns
also apply to descriptors. We could define our hierarchy, set the custom behavior, and so
on. This example follows the OCP, because adding a new type of conversion method would just be about creating a
new class, derived from the base one with the function it needs, without having to modify
the base class itself (to be fair, the previous implementation with decorators was also OCP-
compliant, but there were no classes involved for each transformation mechanism).

Let's take an example where we create a base class that implements the ``__init__()``and
``serialize()`` methods so that we can define the ``LoginEvent`` class simply by deriving
from it, as follows:

.. code-block:: python

    class LoginEvent(BaseEvent):
        username = ShowOriginal()
        password = HideField()
        ip = ShowOriginal()
        timestamp = FormatTime()

Once we achieve this code, the class looks cleaner. It only defines the attributes it needs,
and its logic can be quickly analyzed by looking at the class for each attribute. The base
class will abstract only the common methods, and the class of each event will look simpler
and more compact.

Not only do the classes for each event look simple, but the descriptor itself is very compact
and a lot simpler than the class decorators. The original implementation with class
decorators was good, but descriptors made it even better.
