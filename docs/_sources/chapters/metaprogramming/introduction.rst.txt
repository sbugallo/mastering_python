1. What is metaprogramming
**************************

Maybe there is a good academic definition of metaprogramming that we can cite here, but
this is more about good software craftsmanship than about computer science
theory. This is why we will use the following simple definition:

*"Metaprogramming is a technique of writing computer programs that can treat themselves as data, so they can introspect, generate, and/or modify itself while running."*

Using this definition, we can distinguish between two major approaches to
metaprogramming in Python.

The first approach concentrates on the language's ability to introspect its basic elements,
such as functions, classes, or types, and to create or modify them on the fly. Python really
provides a lot of tools in this area. This feature of the Python language is used by IDEs
(such as PyCharm) to provide real-time code analysis and name suggestions. The easiest
possible metaprogramming tools in Python that utilized language introspection are
decorators that allow for adding extra functionality to the existing functions, methods, or
classes. Next are special methods of classes that allow you to interfere with class instance
process creation. The most powerful are metaclasses, which allow programmers to even
completely redesign Python's implementation of object-oriented programming.

The second approach allows programmers to work directly with code, either in its raw
(plain text) format or in more programmatically accessible abstract syntax tree (AST) form.
This second approach is, of course, more complicated and difficult to work with but allows
for really extraordinary things, such as extending Python's language syntax or even
creating your own **domain-specific language (DSL)**.
