? 1 2 3 4 5 6 7 8 # still running on Python 2.7   from __future__ import
unicode_literals   print '\'xxx\' is unicode?', isinstance('xxx', unicode)
print 'u\'xxx\' is unicode?', isinstance(u'xxx', unicode) print '\'xxx\' is
str?', isinstance('xxx', str) print 'b\'xxx\' is str?', isinstance(b'xxx',
str)

