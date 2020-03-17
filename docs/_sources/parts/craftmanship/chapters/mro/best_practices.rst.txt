4. Best practices
*****************

To avoid all the aforementioned problems, and until Python evolves in this field, we need
to take into consideration the following points:

- **Multiple inheritance should be avoided**: It can be replaced with some design patterns.
- **Super usage has to be consistent**: In a class
  hierarchy, super should be used everywhere or nowhere.
  Mixing super and classic calls is a confusing practice. People tend
  to avoid super to render their code more explicit.
- **Explicitly inherit from an object in Python 3 if you target Python 2 too**: Classes
  without any ancestor specified are recognized as old-style classes in Python 2.
  Mixing old-style classes with new-style classes should be avoided in Python 2.
- **Class hierarchy has to be looked over when a parent class method is called**: To
  avoid any problems, every time a parent class method is called, a quick glance at
  the MRO involved (with __mro__ ) is necessary.