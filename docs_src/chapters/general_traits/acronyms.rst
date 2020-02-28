4. Acronyms to live by
**********************

In this section, we will review some principles that yield some good design ideas. The point is to quickly
relate to good software practices by acronyms that are easy to remember, working as a sort of mnemonic rule.
If you keep these words in mind, you will be able to associate them with good practices more easily, and
finding the right idea behind a particular line of code that you are looking at will be faster.

These are by no means formal or academic definitions, but more like empirical ideas that emerged from years of
working in the software industry. Some of them do appear in books, as they were coined by important authors,
and others have their roots probably in blog posts, papers, or conference talks.

4.1. DRY/OAOO
+++++++++++++

The ideas of **Don't Repeat Yourself (DRY) and Once and Only Once (OAOO)** are closely related, so they were
included together here. They are self-explanatory, you should avoid duplication at all costs.

Things in the code, knowledge, have to be defined only once and in a single place. When you have to make a
change in the code, there should be only one rightful location to modify. Failure to do so is a sign of a
poorly designed system.

Code duplication is a problem that directly impacts maintainability. It is very undesirable to have code
duplication because of its many negative consequences:

- **It's error prone**: When some logic is repeated multiple times throughout the code, and this needs to change, it means we depend on efficiently correcting all the instances with this logic, without forgetting of any of them, because in that case there will be a bug.
- **It's expensive**: Linked to the previous point, making a change in multiple places takes much more time (development and testing effort) than if it was defined only once. This will slow the team down.
- **It's unreliable**: Also linked to the first point, when multiple places need to be changed for a single change in the context, you rely on the person who wrote the code to remember all the instances where the modification has to be made. There is no single source of truth.

Duplication is often caused by ignoring (or forgetting) that code represents knowledge. By giving meaning to
certain parts of the code, we are identifying and labeling that knowledge.

Let's see what this means with an example. Imagine that, in a study center, students are ranked by the
following criteria: 11 points per exam passed, minus five points per exam failed, and minus two per year in
the institution. The following is not actual code, but just a representation of how this might be scattered in
a real code base:

.. code-block:: python

    def process_students_list(students):
        # do some processing...
        students_ranking = sorted(students, key=lambda s: s.passed * 11 - s.failed * 5 - s.years * 2)

        # more processing
        for student in students_ranking:
            print(f"Name: {student.name}, Score: {student.passed * 11 - student.failed * 5 - student.years * 2}")

Notice how the lambda which is in the key of the sorted function represents some valid knowledge from the
domain problem, yet it doesn't reflect it (it doesn't have a name, a proper and rightful location, there is no
meaning assigned to that code, nothing). This lack of meaning in the code leads to the duplication we find
when the score is printed out while listing the raking.

We should reflect our knowledge of our domain problem in our code, and our code will then be less likely to
suffer from duplication and will be easier to understand:

.. code-block:: python

    def score_for_student(student):
        return student.passed * 11 - student.failed * 5 - student.years * 2

    def process_students_list(students):
        # do some processing...
        students_ranking = sorted(students, key=score_for_student)
        # more processing
        for student in students_ranking:
        print(f"Name: {student.name}, Score: {core_for_student(student)}")

A fair disclaimer: this is just an analysis of one of the traits of code duplication. In reality, there are
more cases, types, and taxonomies of code duplication, entire chapters could be dedicated to this topic, but
here we focus on one particular aspect to make the idea behind the acronym clear.

In this example, we have taken what is probably the simplest approach to eliminating duplication: creating a
function. Depending on the case, the best solution would be different. In some cases, there might be an
entirely new object that has to be created (maybe an entire abstraction was missing). In other cases, we can
eliminate duplication with a context manager. Iterators or generators could also help to avoid repetition in
the code, and decorators will also help.

Unfortunately, there is no general rule or pattern to tell you which of the features of Python are the most
suitable to address code duplication, but hopefully, after seeing the examples, and how the elements of Python
are used, you will be able to develop your own intuition.

4.2. YAGNI
++++++++++

**YAGNI (short for You Ain't Gonna Need It)** is an idea you might want to keep in mind very often when
writing a solution if you do not want to over-engineer it.

We want to be able to easily modify our programs, so we want to make them future-proof. In line with that,
many developers think that they have to anticipate all future requirements and create solutions that are very
complex, and so create abstractions that are hard to read, maintain, and understand. Sometime later, it turns
out that those anticipated requirements do not show up, or they do but in a different way (surprise!), and the
original code that was supposed to handle precisely that does not work. The problem is that now it is even
harder to refactor and extend our programs. What happened was that the original solution did not handle the
original requirements correctly, and neither do the current ones, simply because it is the wrong abstraction.

Having maintainable software is not about anticipating future requirements. It is about writing software that
only addresses current requirements in such a way that it will be possible (and easy) to change later on. In
other words, when designing, make sure that your decisions don't tie you down, and that you will be able to
keep on building, but do not build more than what's necessary.

4.3. KIS
++++++++

**KIS (stands for Keep It Simple)** relates very much to the previous point. When you are designing a software
component, avoid over-engineering it; ask yourself if your solution is the minimal one that fits the problem.

Implement minimal functionality that correctly solves the problem and does not complicate your solution more
than is necessary. Remember: the simpler the design, the more maintainable it will be.

This design principle is an idea we will want to keep in mind at all levels of abstraction, whether we are
thinking of a high-level design, or addressing a particular line of code.

At a high-level, think on the components we are creating. Do we really need all of them? Does this module a
ctually require being utterly extensible right now? Emphasize the last part—maybe we want to make that
component extensible, but now is not the right time, or it is not appropriate to do so because we still do not
have enough information to create the proper abstractions, and trying to come up with generic interfaces at
this point will only lead to even worse problems.

In terms of code, keeping it simple usually means using the smallest data structure that fits the problem.
You will most likely find it in the standard library.

Sometimes, we might over-complicate code, creating more functions or methods than what's necessary. The
following class creates a namespace from a set of keyword arguments that have been provided, but it has a
rather complicated code interface:

.. code-block:: python

    class ComplicatedNamespace:
        """An convoluted example of initializing an object with some
        properties."""

        ACCEPTED_VALUES = ("id_", "user", "location")

        @classmethod
        def init_with_data(cls, **data):
            instance = cls()
            for key, value in data.items():
                if key in cls.ACCEPTED_VALUES:
                    setattr(instance, key, value)
            return instance

Having an extra class method for initializing the object doesn't seem really necessary. Then, the iteration,
and the call to setattr inside it, make things even more strange, and the interface that is presented to the
user is not very clear:

.. code-block:: python

    >>> cn = ComplicatedNamespace.init_with_data(
    ...
    id_=42, user="root", location="127.0.0.1", extra="excluded"
    ... )
    >>> cn.id_, cn.user, cn.location
    (42, 'root', '127.0.0.1')
    >>> hasattr(cn, "extra")
    False

The user has to know of the existence of this other method, which is not convenient. It would be better to
keep it simple, and just initialize the object as we initialize any other object in Python (after all, there
is a method for that) with the ``__init__`` method:

.. code-block:: python

    class Namespace:
        """Create an object from keyword arguments."""

        ACCEPTED_VALUES = ("id_", "user", "location")

        def __init__(self, **data):
            accepted_data = {k: v for k, v in data.items() if k in self.ACCEPTED_VALUES}
            self.__dict__.update(accepted_data)

Remember the zen of Python: simple is better than complex.

4.4. EAFP/LBYL
+++++++++++++++

**EAFP (stands for Easier to Ask Forgiveness than Permission), while LBYL (stands for Look Before You Leap)**.

The idea of EAFP is that we write our code so that it performs an action directly, and then we take care of
the consequences later in case it doesn't work. Typically, this means try running some code, expecting it to
work, but catching an exception if it doesn't, and then handling the corrective code on the except block.

This is the opposite of LBYL. As its name says, in the look before you leap approach, we first check what we
are about to use. For example, we might want to check if a file is available before trying to operate with it:

.. code-block:: python

    if os.path.exists(filename):
        with open(filename) as f:
        ...

This might be good for other programming languages, but it is not the Pythonic way of writing code. Python was
built with ideas such as EAFP, and it encourages you to follow them (remember, explicit is better than
implicit). This code would instead be rewritten like this:

.. code-block:: python

    try:
    with open(filename) as f:
        ...
    except FileNotFoundError as e:
        logger.error(e)

.. note:: Prefer EAFP over LBYL.
