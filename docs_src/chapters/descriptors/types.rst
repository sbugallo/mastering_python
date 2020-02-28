2. Types of descriptors
***********************

Based on the methods we have just explored, we can make an important distinction among
descriptors in terms of how they work. Understanding this distinction plays an important
role in working effectively with descriptors, and will also help to avoid caveats or common
errors at runtime.

If a descriptor implements the ``__set__`` or ``__delete__`` methods, it is called a **data
descriptor**. Otherwise, a descriptor that solely implements ``__get__`` is a **non-data
descriptor**. Notice that ``__set_name__`` does not affect this classification at all.

When trying to resolve an attribute of an object, a data descriptor will always take
precedence over the dictionary of the object, whereas a non-data descriptor will not. This
means that in a non-data descriptor if the object has a key on its dictionary with the same
name as the descriptor, this one will always be called, and the descriptor itself will never
run. Conversely, in a data descriptor, even if there is a key in the dictionary with the same
name as the descriptor, this one will never be used since the descriptor itself will always
end up being called.

The following two sections explain this in more detail, with examples, in order to get a
deeper idea of what to expect from each type of descriptor.

2.1. Non-data descriptors
+++++++++++++++++++++++++

We will start with a descriptor that only implements the ``__get__`` method, and see how
it is used:

.. code-block:: python

    class NonDataDescriptor:
        def __get__(self, instance, owner):
            if instance is None:
                return self
            return 42

    class ClientClass:
        descriptor = NonDataDescriptor()

As usual, if we ask for the descriptor, we get the result of its ``__get__`` method:

.. code-block:: python

    >>> client = ClientClass()
    >>> client.descriptor
    42

But if we change the descriptor attribute to something else, we lose access to this value,
and get what was assigned to it instead:

.. code-block:: python

    >>> client.descriptor = 43
    >>> client.descriptor
    43

Now, if we delete the descriptor, and ask for it again, let's see what we get:

.. code-block:: python

    >>> del client.descriptor
    >>> client.descriptor
    42

Let's rewind what just happened. When we first created the client object, the
descriptor attribute lay in the class, not the instance, so if we ask for the dictionary of the
client object, it will be empty:

.. code-block:: python

    >>> vars(client)
    {}

And then, when we request the ``.descriptor`` attribute, it doesn't find any key in
``client.__dict__`` named "descriptor", so it goes to the class, where it will find it, but
only as a descriptor, hence why it returns the result of the ``__get__`` method.

But then, we change the value of the ``.descriptor`` attribute to something else, and what
this does is set this into the dictionary of the instance, meaning that this time it won't be
empty:

.. code-block:: python

    >>> client.descriptor = 99
    >>> vars(client)
    {'descriptor': 99}

So, when we ask for the ``.descriptor`` attribute here, it will look for it in the object (and
this time it will find it, because there is a key named descriptor in the ``__dict__`` attribute
of the object, as the vars result is showing us), and return it without having to look for it in
the class. For this reason, the descriptor protocol is never invoked, and the next time we ask
for this attribute, it will instead return the value we have overridden it with (99).

Afterward, we delete this attribute by calling ``del``, and what this does is remove the key
"descriptor" from the dictionary of the object, leaving us back in the first scenario, where
it's going to default to the class where the descriptor protocol will be activated:

.. code-block:: python

    >>> del client.descriptor
    >>> vars(client)
    {}
    >>> client.descriptor
    42

This means that if we set the attribute of the descriptor to something else, we might be
accidentally breaking it. Why? Because the descriptor doesn't handle the delete action
(some of them don't need to).

This is called a non-data descriptor because it doesn't implement the ``__set__`` magic
method, as we will see in the next example.

2.2. Data descriptors
+++++++++++++++++++++

Now, let's look at the difference of using a data descriptor. For this, we are going to create
another simple descriptor that implements the ``__set__`` method:

.. code-block:: python

    class DataDescriptor:
        def __get__(self, instance, owner):
            if instance is None:
                return self
            return 42

        def __set__(self, instance, value):
            logger.debug("setting %s.descriptor to %s", instance, value)
            instance.__dict__["descriptor"] = value

    class ClientClass:
        descriptor = DataDescriptor()

Let's see what the value of the descriptor returns:

.. code-block:: python

    >>> client = ClientClass()
    >>> client.descriptor
    42

Now, let's try to change this value to something else, and see what it returns instead:

.. code-block:: python

    >>> client.descriptor = 99
    >>> client.descriptor
    42

The value returned by the descriptor didn't change. But when we assign a different value
to it, it must be set to the dictionary of the object (as it was previously):

.. code-block:: python

    >>> vars(client)
    {'descriptor': 99}
    >>> client.__dict__["descriptor"]
    99

So, the ``__set__()`` method was called, and indeed it did set the value to the dictionary of
the object, only this time, when we request this attribute, instead of using the ``__dict__``
attribute of the dictionary, the descriptor takes precedence (because it's an overriding
descriptor ).

One more thing: deleting the attribute will not work anymore:

.. code-block:: python

    >>> del client.descriptor
    Traceback (most recent call last):
    ...
    AttributeError: __delete__

The reason is as follows: given that now, the descriptor always takes place, calling
``del`` on an object doesn't try to delete the attribute from the dictionary (``__dict__``) of the
object, but instead it tries to call the ``__delete__()`` method of the descriptor (which is
not implemented in this example, hence the attribute error).

This is the difference between data and non-data descriptors. If the descriptor implements
``__set__()``, then it will always take precedence, no matter what attributes are present in
the dictionary of the object. If this method is not implemented, then the dictionary will be
looked up first, and then the descriptor will run.

An interesting observation you might have noticed is this line on the set method:
``instance.__dict__["descriptor"] = value``. There are a lot of things to question about that line, but let's
break it down into parts.

First, why is it altering just the name of a "descriptor" attribute? This is just a
simplification for this example, but, as it transpires when working with descriptors, it
doesn't know at this point the name of the parameter it was assigned to, so we just used the
one from the example, knowing that it was going to be "descriptor".

In a real example, you would do one of two things: either receive the name as a parameter
and store it internally in the ``init`` method, so that this one will just use the internal
attribute, or, even better, use the ``__set_name__`` method.

Why is it accessing the ``__dict__`` attribute of the instance directly? Another good question,
which also has at least two explanations. First, you might be thinking why not just do the
following: ``setattr(instance, "descriptor", value)``. Remember that this method (``__set__``)
is called when we try to assign something to the
attribute that is a descriptor. So, using ``setattr()`` will call this descriptor again,
which, in turn, will call it again, and so on and so forth. This will end up in an infinite
recursion.

.. note:: Do not use ``setattr()`` or the assignment expression directly on the descriptor inside the ``__set__`` method because that will trigger an infinite recursion.

Why, then, is the descriptor not able to book-keep the values of the properties for all of its
objects?

The client class already has a reference to the descriptor. If we add a reference from the
descriptor to the client object, we are creating circular dependencies, and these objects
will never be garbage-collected. Since they are pointing at each other, their reference counts
will never drop below the threshold for removal.

A possible alternative here is to use weak references, with the ``weakref`` module, and create
a weak reference key dictionary if we want to do that. This implementation is explained
later on, but we prefer to use this idiom, since it is fairly common and accepted when writing descriptors.
