1. PEP 8
********

PEP 8 (`http://www.python.org/dev/peps/pep-0008 <http://www.python.org/dev/peps/pep-0008>`_)
provides a style guide for writing
Python code. Besides some basic rules, such as indentation, maximum line length, and
other details concerning the code layout, PEP 8 also provides a section on naming
conventions that most of the code bases follow.

This section provides only a quick summary of PEP 8, and a handy naming guide for each
kind of Python syntax element. You should still consider reading the PEP 8 document as
mandatory.

1.1. Why and when to follow PEP 8?
++++++++++++++++++++++++++++++++++

If you are creating a new software package that is intended to be open sourced, you should
always follow PEP 8 because it is a widely accepted standard and is used in most of the
open source projects written in Python. If you want to foster any collaboration with other
programmers, then you should definitely stick to PEP 8, even if you have different views on
the best code style guidelines. Doing so has the benefit of making it a lot easier for other
developers to jump straight into your project. Code will be easier to read for newcomers
because it will be consistent in style with most of the other Python open source packages.

Also, starting with full PEP 8 compliance saves you time and trouble in the future. If you
want to release your code to the public, you will eventually face suggestions from fellow
programmers to switch to PEP 8. Arguments as to whether it is really necessary to do so for
a particular project tend to be never-ending flame wars that are impossible to win. This is
the sad truth, but you may be eventually forced to be consistent with it or risk losing
valuable contributors.

Also, restyling of the whole project's code base if it is in a mature state of development
might require a tremendous amount of work. In some cases, such restyling might require
changing almost every line of code. While most of the changes can be automated
(indentation, newlines, and trailing whitespaces), such massive code overhaul usually
introduces a lot of conflicts in every version control workflow that is based on branching. It
is also very hard to review so many changes at once. These are the reasons why many open
source projects have a rule that style-fixing changes should always be included in separate
pull/merge requests or patches that do not affect any feature or bug.

1.2. Team-specific style guidelines
+++++++++++++++++++++++++++++++++++

Despite providing a comprehensive set of style guidelines, PEP 8 still leaves some freedom
for the developers. Especially in terms of nested data literals and multiline function calls
that require long lists of arguments. Some teams may decide that they require additional
styling rules and the best option is to formalize them in some kind of document that is
available for every team member.

Also, in some situations, it may be impossible or economically infeasible to be strictly
consistent with PEP 8 in some old projects that had no style guide defined. Such projects
will still benefit from formalization of the actual coding conventions even if they do not
reflect the official set of PEP 8 rules. Remember, what is more important than consistency
with PEP 8 is consistency within the project. If rules are formalized and available as a
reference for every programmer, then it is way easier to keep consistency within a project
and organization.