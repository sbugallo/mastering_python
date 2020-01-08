4. Decorators and separation of concerns
****************************************

The last point on the previous list is so important that it deserves a section of its own. We
have already explored the idea of reusing code and noticed that a key element of reusing
code is having components that are cohesive. This means that they should have the
minimum level of responsibility: do one thing, one thing only, and do it well. The smaller
our components, the more reusable, and the more they can be applied in a different context
without carrying extra behavior that will cause coupling and dependencies, which will
make the software rigid.

To show you what this means, let's reprise one of the decorators that we used in a previous
example. We created a decorator that traced the execution of certain functions with code
similar to the following:

.. code-block:: python

    def traced_function(function):

        @functools.wraps(function)
        def wrapped(*args, **kwargs):
            logger.info("started execution of %s", function.__qualname__)
            start_time = time.time()
            result = function(*args, **kwargs)
            logger.info(
                "function %s took %.2fs",
                function.__qualname__,
                time.time() - start_time
            )
            return result

        return wrapped

Now, this decorator, while it works, has a problem: it is doing more than one thing. It logs
that a particular function was just invoked, and also logs how much time it took to run.
Every time we use this decorator, we are carrying these two responsibilities, even if we only
wanted one of them.

This should be broken down into smaller decorators, each one with a more specific and
limited responsibility:

.. code-block:: python

    def log_execution(function):
        @wraps(function)
        def wrapped(*args, **kwargs):
            logger.info("started execution of %s", function.__qualname__)
            return function(*kwargs, **kwargs)
        return wrapped

    def measure_time(function):
        @wraps(function)
        def wrapped(*args, **kwargs):
            start_time = time.time()
            result = function(*args, **kwargs)
            logger.info("function %s took %.2f", function.__qualname__,
            time.time() - start_time)
            return result
        return wrapped

Notice that the same functionality that we had previously can be achieved by simply combining both of them:

.. code-block:: python

    @measure_time
    @log_execution
    def operation():
        ....

Notice how the order in which the decorators are applied is also important.

.. note:: Do not place more than one responsibility in a decorator. The SRP applies to decorators as well.