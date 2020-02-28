5. Analyzing good decorators
****************************

As a closing note for this chapter, let's review some examples of good decorators and how
they are used both in Python itself, as well as in popular libraries. The idea is to get
guidelines on how good decorators are created.

Before jumping into examples, let's first identify traits that good decorators should have:

- **Encapsulation, or separation of concerns**: A good decorator should effectively separate different responsibilities between what it does and what it is decorating. It cannot be a leaky abstraction, meaning that a client of the decorator should only invoke it in black box mode, without knowing how it is actually implementing its logic.
- **Orthogonality**: What the decorator does should be independent, and as decoupled as possible from the object it is decorating.
- **Reusability**: It is desirable that the decorator can be applied to multiple types, and not that it just appears on one instance of one function, because that means that it could just have been a function instead. It has to be generic enough.

A nice example of decorators can be found in the Celery project, where a task is defined by
applying the decorator of the task from the application to a function:

.. code-block:: python

    @app.task
    def mytask():
        ....

One of the reasons why this is a good decorator is because it is very good at
something: encapsulation. The user of the library only needs to define the function body
and the decorator will convert that into a task automatically. The ``@app.task`` decorator
surely wraps a lot of logic and code, but none of that is relevant to the body of
``mytask()`` . It is complete encapsulation and separation of concerns—nobody will have to
take a look at what that decorator does, so it is a correct abstraction that does not leak any
details.

Another common use of decorators is in web frameworks (Pyramid, Flask, and Sanic, just
to name a few), on which the handlers for views are registered to the URLs through
decorators:

.. code-block:: python

    @route("/", method=["GET"])
    def view_handler(request):
        ...

These sorts of decorator have the same considerations as before; they also provide total
encapsulation because a user of the web framework rarely (if ever) needs to know what
the ``@route`` decorator is doing. In this case, we know that the decorator is doing
something more, such as registering these functions to a mapper to the URL, and also that it
is changing the signature of the original function to provide us with a nicer interface that
receives a request object with all the information already set.

The previous two examples are enough to make us notice something else about this use of
decorators. They conform to an API. These libraries of frameworks are exposing their
functionality to users through decorators, and it turns out that decorators are an excellent
way of defining a clean programming interface.

This is probably the best way we should think about to decorators. Much like in the
example of the class decorator that tells us how the attributes of the event are going to be
handled, a good decorator should provide a clean interface so that users of the code know
what to expect from the decorator, without needing to know how it works, or any of its
details for that matter.