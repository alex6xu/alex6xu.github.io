from contextlib import contextmanager @contextmanager def tag(name): print
"<%s>" % name yield print "" % name >>> with tag("h1"): ... print "foo" ...

#  foo

