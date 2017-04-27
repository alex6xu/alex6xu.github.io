1\. 使用标准的数据库优化技术： 在进行Django数据库访问性能优化之前，首先应该使用标准的数据库技术对其进行优化，比如给字段加索引，通过使用
django.db.models.Field.db_index
来给一个Django模型类的字段加索引，设置这个属性字段的Field.db_index=True。 注：django对model中的fk和unique =
True的字段将自动创建索引。 2\. 理解Django中QuerySet的工作机制对数据库访问优化至关重要：
QuerySet是懒加载的，它只有在需要的时候才会被执行，并且会将执行的结果保存在内存中。 3\. 理解Django中QuerySet的缓存机制：
QuerySet对调用方法是不执行缓存的。比如下面的两端代码，其中一个会被缓存，另一个不会：    >>> entry =
Entry.objects.get(id=1)    >>> entry.blog # Blog对象会被从数据库查询出来    >>> entry.blog
# 第二次访问的缓存对象，不会再次执行查询 但是对于调用的查询方法，是不会被缓存的：    >>> entry =
Entry.objects.get(id=1)    >>> entry.authors.all() # 第一次会执行查询    >>>
entry.authors.all() # 第二次会再执行一次查询 4\. 使用模板语言中的with标签：
在视图模板中，针对QuerySet对象使用with标签，可以让数据被缓存起来使用。 5\. 使用iterator()方法：
对于缓存的QuerySet使用iterator()方法。 6\. 将查询计算操作放在数据库中完成，不要在Python代码中完成。 1)
使用filter,exclude完成查询过滤； 2) F()查询表达式； 3) 使用聚合函数来完成数据库聚合操作。 7\.
使用QuerySet.extra()明确的指出要查询的字段。 8\. 对于复杂的数据库查询操作，使用原生SQL实现。 9\. 尽量一次查询出所有需要的信息。
10\. 只查询需要的数据： 1) 某些情况下，只使用 QuerySet.values()和
values_list()方法，查询出符合条件的结果集而不是完整的对象结果集； 2) 某些情况下，只使用 QuerySet.defer() 和
only()过滤数据。 11\. 如果只是查询集合的数量，使用QuerySet.count()函数，而不是len(QuerySet)； 12\.
如果想知道某个记录是否包含在某个结果集中，使用 QuerySet.exists()函数； 13\. 避免过多的使用 count() 和 exists()
函数； 14\. 对于批量更新和删除操作使用 QuerySet.update() 和 QuerySet.delete()； 15\. 理解
QuerySet.select_related() 方法：
select_related()会在查询过程中尽量深入的查询关联数据，这样在需要查询大量外键的数据时非常有用，如：
>>>e=Entry.objects.get(id=5) #这部操作会查询数据库    >>>b=e.blog #该操作会再次查询数据库
而采用select_related()查询的效果是：    >>>e=Entry.objects.select_related().get(id=5)
#这步操作会查询数据库    >>>b=e.blog #不会再次查询数据库 16\.
如果需要查询对象的外键，则使用外键字段而不是使用关联的对象的主键，比如： >>>entry.blog_id #应该使用这种方式
>>>entry.blog.id #不要使用这种方式

