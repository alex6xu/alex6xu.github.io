Python代码   class Blog(models.Model):       name =
models.CharField(max_length=100)       tagline = models.TextField()
def __unicode__(self):           return self.name      class
Author(models.Model):       name = models.CharField(max_length=50)       email
= models.EmailField()          def __unicode__(self):           return
self.name      class Entry(models.Model):       blog = models.ForeignKey(Blog)
headline = models.CharField(max_length=255)       body_text =
models.TextField()       pub_date = models.DateTimeField()       authors =
models.ManyToManyField(Author)          def __unicode__(self):
return self.headline   这是model，有blog，author，以及entry；其中entry分别与blog与author表关联，e
ntry与blog表是通过外键(models.ForeignKey())相连，属于一对多的关系，即一个entry对应多个blog，entry与author是
多对多的关系，通过modles.ManyToManyField()实现。  一、插入数据库，用save()方法实现，如下：  >>> from
mysite.blog.models import Blog  >>> b = Blog(name='Beatles Blog', tagline='All
the latest Beatles news.')  >>> b.save()  二、更新数据库，也用save()方法实现，如下：  >> b5.name
= 'New name'  >> b5.save()  保存外键和多对多关系的字段，如下例子：  更新外键字段和普通的字段一样，只要指定一个对象的正确类型。
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")  >>> entry.blog =
cheese_blog  >>> entry.save()  更新多对多字段时又一点不太一样，使用add()方法添加相关联的字段的值。  >> joe =
Author.objects.create(name="Joe")  >> entry.authors.add(joe)  三、检索对象  >>>
Blog.objects    >>> b = Blog(name='Foo', tagline='Bar')  >>> b.objects
Traceback:      ...  AttributeError: "Manager isn't accessible via Blog
instances."  1、检索所有的对象  >>> all_entries = Entry.objects.all()
使用all()方法返回数据库中的所有对象。  2、检索特定的对象  使用以下两个方法：  fileter(**kwargs)
返回一个与参数匹配的QuerySet,相当于等于(=).  exclude(**kwargs)
返回一个与参数不匹配的QuerySet,相当于不等于(!=)。  Entry.objects.filter(pub_date__year=2006)
不使用Entry.objects.all().filter(pub_date__year=2006),虽然也能运行，all()最好再获取所有的对象时使用。
上面的例子等同于的sql语句：  slect * from entry where pub_date_year='2006'  链接过滤器：  >>>
Entry.objects.filter(  ...     headline__startswith='What'  ... ).exclude(
...     pub_date__gte=datetime.now()  ... ).filter(  ...
pub_date__gte=datetime(2005, 1, 1)  ... )  最后返回的QuerySet是headline like 'What%'
and put_date2005-01-01  另外一种方法：  >> q1 =
Entry.objects.filter(headline__startswith="What")  >> q2 =
q1.exclude(pub_date__gte=datetime.now())  >> q3 =
q1.filter(pub_date__gte=datetime.now())  这种方法的好处是可以对q1进行重用。  QuerySet是延迟加载
只在使用的时候才会去访问数据库，如下：  >>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.now())  >>> q =
q.exclude(body_text__icontains="food")  >>> print q  在print q时才会访问数据库。
其他的QuerySet方法  >>> Entry.objects.all()[:5]  这是查找前5个entry表里的数据  >>>
Entry.objects.all()[5:10]  这是查找从第5个到第10个之间的数据。  >>> Entry.objects.all()[:10:2]
这是查询从第0个开始到第10个，步长为2的数据。  >>> Entry.objects.order_by('headline')[0]
这是取按headline字段排序后的第一个对象。  >>> Entry.objects.order_by('headline')[0:1].get()
这和上面的等同的。  >>> Entry.objects.filter(pub_date__lte='2006-01-01')  等同于SELECT *
FROM blog_entry WHERE pub_date <= '2006-01-01';  >>>
Entry.objects.get(headline__exact="Man bites dog")  等同于SELECT ... WHERE
headline = 'Man bites dog';  >>> Blog.objects.get(id__exact=14)  # Explicit
form  >>> Blog.objects.get(id=14)         # __exact is implied
这两种方式是等同的，都是查找id=14的对象。  >>> Blog.objects.get(name__iexact="beatles blog")
查找name="beatles blog"的对象，不去饭大小写。
Entry.objects.get(headline__contains='Lennon')  等同于SELECT ... WHERE headline
LIKE '%Lennon%';  startswith 等同于sql语句中的 name like 'Lennon%',
endswith等同于sql语句中的 name like '%Lennon'.  >>>
Entry.objects.filter(blog__name__exact='Beatles Blog')
查找entry表中外键关系blog_name='Beatles Blog'的Entry对象。  >>>
Blog.objects.filter(entry__headline__contains='Lennon')
查找blog表中外键关系entry表中的headline字段中包含Lennon的blog数据。
Blog.objects.filter(entry__author__name='Lennon')
查找blog表中外键关系entry表中的author字段中包含Lennon的blog数据。
Blog.objects.filter(entry__author__name__isnull=True)  Blog.objects.filter(ent
ry__author__isnull=False,entry__author__name__isnull=True)
查询的是author_name为null的值  Blog.objects.filter(entry__headline__contains='Lennon'
,entry__pub_date__year=2008)
Blog.objects.filter(entry__headline__contains='Lennon').filter(
entry__pub_date__year=2008)
这两种查询在某些情况下是相同的，某些情况下是不同的。第一种是限制所有的blog数据的，而第二种情况则是第一个filter是
限制blog的，而第二个filter则是限制entry的  >>> Blog.objects.get(id__exact=14) # Explicit
form  >>> Blog.objects.get(id=14) # __exact is implied  >>>
Blog.objects.get(pk=14) # pk implies id__exact  等同于select * from where id=14
# Get blogs entries with id 1, 4 and 7  >>>
Blog.objects.filter(pk__in=[1,4,7])  等同于select * from where id in{1,4,7}  #
Get all blog entries with id > 14  >>> Blog.objects.filter(pk__gt=14)
等同于select * from id>14  >>> Entry.objects.filter(blog__id__exact=3) # Explicit
form  >>> Entry.objects.filter(blog__id=3)        # __exact is implied  >>>
Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact  这三种情况是相同的
>>> Entry.objects.filter(headline__contains='%')  等同于SELECT ... WHERE headline
LIKE '%\%%';  Caching and QuerySets  >>> print [e.headline for e in
Entry.objects.all()]  >>> print [e.pub_date for e in Entry.objects.all()]
应改写为：  >> queryset = Poll.objects.all()  >>> print [p.headline for p in
queryset] # Evaluate the query set.  >>> print [p.pub_date for p in queryset]
# Re-use the cache from the evaluation.、  这样利用缓存，减少访问数据库的次数。  四、用Q对象实现复杂的查询
Q(question__startswith='Who') | Q(question__startswith='What')  等同于WHERE
question LIKE 'Who%' OR question LIKE 'What%'  Poll.objects.get(
Q(question__startswith='Who'),      Q(pub_date=date(2005, 5, 2)) |
Q(pub_date=date(2005, 5, 6))  )  等同于SELECT * from polls WHERE question LIKE
'Who%' AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
Poll.objects.get(      Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5,
6)),      question__startswith='Who')
等同于Poll.objects.get(question__startswith='Who', Q(pub_date=date(2005, 5, 2)) |
Q(pub_date=date(2005, 5, 6)))  五、比较对象  >>> some_entry == other_entry  >>>
some_entry.id == other_entry.id  六、删除
Entry.objects.filter(pub_date__year=2005).delete()  b = Blog.objects.get(pk=1)
# This will delete the Blog and all of its Entry objects.  b.delete()
Entry.objects.all().delete()  删除所有  七、一次更新多个值  # Update all the headlines with
pub_date in 2007.
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the
same')  >>> b = Blog.objects.get(pk=1)  # Change every Entry so that it
belongs to this Blog.  >>> Entry.objects.all().update(blog=b)
如果用save()方法，必须一个一个进行保存，需要对其就行遍历，如下：  for item in my_queryset:      item.save()
关联对象  one-to-many  >>> e = Entry.objects.get(id=2)  >>> e.blog # Returns the
related Blog object.  >>> e = Entry.objects.get(id=2)  >>> e.blog = some_blog
>>> e.save()  >>> e = Entry.objects.get(id=2)  >>> e.blog = None  >>> e.save()
# "UPDATE blog_entry SET blog_id = NULL ...;"  >>> e = Entry.objects.get(id=2)
>>> print e.blog  # Hits the database to retrieve the associated Blog.  >>>
print e.blog  # Doesn't hit the database; uses cached version.  >>> e =
Entry.objects.select_related().get(id=2)  >>> print e.blog  # Doesn't hit the
database; uses cached version.  >>> print e.blog  # Doesn't hit the database;
uses cached version  >>> b = Blog.objects.get(id=1)  >>> b.entry_set.all() #
Returns all Entry objects related to Blog.  # b.entry_set is a Manager that
returns QuerySets.  >>> b.entry_set.filter(headline__contains='Lennon')  >>>
b.entry_set.count()  >>> b = Blog.objects.get(id=1)  >>> b.entries.all() #
Returns all Entry objects related to Blog.  # b.entries is a Manager that
returns QuerySets.  >>> b.entries.filter(headline__contains='Lennon')  >>>
b.entries.count()  You cannot access a reverse ForeignKey Manager from the
class; it must be accessed from an instance:  >>> Blog.entry_set  add(obj1,
obj2, ...)      Adds the specified model objects to the related object set.
create(**kwargs)      Creates a new object, saves it and puts it in the
related object set. Returns the newly created object.  remove(obj1, obj2, ...)
Removes the specified model objects from the related object set.  clear()
Removes all objects from the related object set.       many-to-many类型：  e =
Entry.objects.get(id=3)  e.authors.all() # Returns all Author objects for this
Entry.  e.authors.count()  e.authors.filter(name__contains='John')  a =
Author.objects.get(id=5)  a.entry_set.all() # Returns all Entry objects for
this Author.  one-to-one 类型：  class EntryDetail(models.Model):      entry =
models.OneToOneField(Entry)      details = models.TextField()  ed =
EntryDetail.objects.get(id=2)  ed.entry # Returns the related Entry object
使用sql语句进行查询：  def my_custom_sql(self):      from django.db import connection
cursor = connection.cursor()      cursor.execute("SELECT foo FROM bar WHERE
baz = %s", [self.baz])      row = cursor.fetchone()      return row

