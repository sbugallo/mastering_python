7. Final remarks on good practices for software design
******************************************************

A good software design involves a combination of following good practices of software engineering and taking
advantage of most of the features of the language. There is a great value in using everything that Python has
to offer, but there is also a great risk of abusing this and trying to fit complex features into simple
designs.

In addition to this general principle, it would be good to add some final recommendations.

7.1. Orthogonality in software
+++++++++++++++++++++++++++++++

This word is very general and can have multiple meanings or interpretations. In math, orthogonal means that
two elements are independent. If two vectors are orthogonal, their scalar product is zero. It also means they
are not related at all: a change in one of them doesn't affect the other one at all. That's the way we should
think about our software.

Changing a module, class, or function should have no impact on the outside world to that component that is
being modified. This is of course highly desirable, but not always possible. But even for cases where it's not
possible, a good design will try to minimize the impact as much as possible. We have seen ideas such as
separation of concerns, cohesion, and isolation of components.

In terms of the runtime structure of software, orthogonality can be interpreted as the fact that makes changes
(or side-effects) local. This means, for instance, that calling a method on an object should not alter the
internal state of other (unrelated) objects. We have already (and will continue to do so) emphasized the
importance of minimizing side-effects in our code.

In the example with the mixin class, we created a tokenizer object that returned an iterable. The fact that
the ``__iter__`` method returned a new generator increases the chances that all three classes (the base,
the mixing, and the concrete class) are orthogonal. If this had returned something in concrete (a list, let's
say), this would have created a dependency on the rest of the classes, because when we changed the list to
something else, we might have needed to update other parts of the code, revealing that the classes were not as
independent as they should be.

Let's show you a quick example. Python allows passing functions by parameter because they are just regular
objects. We can use this feature to achieve some orthogonality. We have a function that calculates a price,
including taxes and discounts, but afterward we want to format the final price that's obtained:

.. code-block:: python

    def calculate_price(base_price: float, tax: float, discount: float) ->
        return (base_price * (1 + tax)) * (1 - discount)

    def show_price(price: float) -> str:
        return "$ {0:,.2f}".format(price)

    def str_final_price(base_price: float, tax: float, discount: float, fmt_function=str) -> str:
        return fmt_function(calculate_price(base_price, tax, discount))

Notice that the top-level function is composing two orthogonal functions. One thing to notice is how we
calculate the price, which is how the other one is going to be represented. Changing one does not change the
other. If we don't pass anything in particular, it will use string conversion as the default representation
function, and if we choose to pass a custom function, the resulting string will change. However, changes in
``show_price`` do not affect ``calculate_price`` . We can make changes to either function, knowing that the
other one will remain as it was:

.. code-block:: python

    >>> str_final_price(10, 0.2, 0.5)
    '6.0'
    >>> str_final_price(1000, 0.2, 0)
    '1200.0'
    >>> str_final_price(1000, 0.2, 0.1, fmt_function=show_price)
    '$ 1,080.00'

There is an interesting quality aspect that relates to orthogonality. If two parts of the code are orthogonal,
it means one can change without affecting the other. This implies that the part that changed has unit tests
that are also orthogonal to the unit tests of the rest of the application. Under this assumption, if those
tests pass, we can assume (up to a certain degree) that the application is correct without needing full
regression testing.

More broadly, orthogonality can be thought of in terms of features. Two functionalities of the application can
be totally independent so that they can be tested and released without having to worry that one might break
the other (or the rest of the code, for that matter).

Imagine that the project requires a new authentication mechanism (``oauth2``, let's say, but just for the sake
of the example), and at the same time another team is also working on a new report. Unless there is something
fundamentally wrong in that system, neither of those features should impact the other. Regardless of which one
of those gets merged first, the other one should not be affected at all.

7.2. Structuring the code
+++++++++++++++++++++++++

The way code is organized also impacts the performance of the team and its maintainability.

In particular, having large files with lots of definitions (classes, functions, constants, and so on) is a bad
practice and should be discouraged. This doesn't mean going to the extreme of placing one definition per file,
but a good code base will structure and arrange components by similarity.

Luckily, most of the time, changing a large file into smaller ones is not a hard task in Python. Even if
multiple other parts of the code depend on definitions made on that file, this can be broken down into a
package, and will maintain total compatibility. The idea would be to create a new directory with a
``__init__.py`` file on it (this will make it a Python package). Alongside this file, we will have multiple
files with all the particular definitions each one requires (fewer functions and classes grouped by a certain
criterion). Then, the ``__init__.py`` file will import from all the other files the definitions it previously
had (which is what guarantees its compatibility). Additionally, these definitions can be mentioned in the
``__all__`` variable of the module to make them exportable.

There are many advantages of this. Other than the fact that each file will be easier to navigate, and things
will be easier to find, we could argue that it will be more efficient because of the following reasons:

- It contains fewer objects to parse and load into memory when the module is imported
- The module itself will probably be importing fewer modules because it needs fewer dependencies, like before

It also helps to have a convention for the project. For example, instead of placing constants in all of the
files, we can create a file specific to the constant values to be used in the project, and import it from
there: ``from myproject.constants import CONNECTION_TIMEOUT``. Centralizing information like this makes it
easier to reuse code and helps to avoid inadvertent duplication.

More details about separating modules and creating Python packages will be discussed in
Chapter 10, Clean Architecture, when we explore this in the context of software architecture.
