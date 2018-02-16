title: QuickNote Descriptor in Python
date: 2014-4-12
tags: Python
---

TL;DR: Descriptor in Python is just getter/setter

Three ways to implement descriptor:

  1. Use a class that implement magic method `__get__`, `__set__`, `__delete__`
  2. Use builtin function `property(fget=None, fset=None, fdel=None, doc=None)`
  3. Use decorator form of `property`, like `@property def attr(): ...`, `@attr.setter`, `@attr.deleter`

Make sure all three methods are done at class level rather than at instance level.
(That is, write `my_attr = property(...)` under `class MyClass(object):` statement,
but not `self.my_attr = property(..)` in `__init__`)

Because Python's MRO is:

  1. find `attr` in `instance.__dict__` (instance level)
  2. if not found, find `attr` in `instance.__class__.__dict__`. If found, try return `attr.__get__`, otherwise return `attr` (class level)
  3. if not found, repeat step 2 on `instance.__class__.__base__` until `attr` found or no base class found
  4. if not found, try returning computed `attr` if `__getattr__` is defined

So the descriptor magic is done at class level

Pitfall:

Because descriptor dwells at class level, if one uses descriptor class implementation, all instances may share one common variable.

To fix that:
  1. Hoard one hidden variable in the instance
  2. Use a dictionary in descriptor class to store info of different instances.

The second solution needs the class hashable. (So a meta class is needed to guarantee hashability)


Personal view: To maintain Python's `one simple way to work` Pythonists just introduced much more `complexity`.

Reference:
http://www.ibm.com/developerworks/library/os-pythondescriptors/
http://nbviewer.ipython.org/urls/gist.github.com/ChrisBeaumont/5758381/raw/descriptor_writeup.ipynb
http://docs.python.org/2/howto/descriptor.html
