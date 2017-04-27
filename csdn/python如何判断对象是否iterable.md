1 通常判断 hasattr(myObj, '__iter__') 2， try: some_object_iterator =
iter(some_object) except TypeError, te: print some_object, 'is not iterable'
3， try: _ = (e for e in my_object) except TypeError: print my_object, 'is not
iterable' 4 import collections if isinstance(e, collections.Iterable): # e is
iterable

