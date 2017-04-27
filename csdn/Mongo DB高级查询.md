常用命令 > show dbs    -- 查看数据库列表 > use admin
--创建admin数据库，如果存在admin数据库则使用admin数据库 > db   ---显示当前使用的数据库名称 > db.getName()
---显示当前使用的数据库名称 > db.dropDatabase()  --删当前使用的数据库 > db.repairDatabase()
--修复当前数据库 > db.version()   --当前数据库版本 > db.getMongo()  --查看当前数据库的链接机器地址  >
db.stats() 显示当前数据库状态，包含数据库名称，集合个数，当前数据库大小 ... > db.getCollectionNames()
--查看数据库中有那些个集合（表） > show collections    --查看数据库中有那些个集合（表） > db.person.drop()
--删除当前集合（表）person
MongoDB接入Javascrip风格语法，for，while，next，hasNext，forEach，toArray，findOne，limit
Note: 1、p={name:"张龙豪"，age:18}
这个不是规范的json格式，使用这种类似javascript语法，对象会自动补全为规范的json格式{"name":"张龙豪","age":18}.
2、我在这里插入2条数据，一条是insert语法，一条是save语法，这里的insert与save是一样的功能。
Note:这里我主要用啦一个for语法，循环插入啦4条数据，是不是有点小激动。哈哈，mongodb就是这么任性。 Note：这里是吧查询出来的
1、while：作为程序员应该都不陌生他是个循环。 2、hasNext： cursor集合遍历，是否还有数据。 3、printjson：输出集合中的文档
4、next：当前文档，并向下遍历。 Note：forEach循环输入，必须定义一个函数供每个游标元素调用。 Note：游标也可以当作数组来用。
Note：游标转换为真实的数组类型，使用。 Note：findOne，返回结果集中的第一条数据。limit(3)，返回结果集中的前三条数据。
MongoDB中的高级查询
面向文档的NoSql数据库重要解决的问题不是高性能的并发读写问题，而是保证海量数据存储的同时，具有比一般数据库更加良好的查询性能。 Note:条件操作符号：
> 、 < 、 >= 、<=  1、 $gt //大于 > ，$lt //小于 < ，$gte //大于等于 >= ,$lte //小于等于
2、{"filed":{$op,value}}  //filed字段 ，$op条件操作符号，value值。 3、{"age":{$gt:1}} :
person集合中年龄大于1的所有数据文档  Note:$all匹配所有，类似t-sql中的in，但是t-
sql中的in是满足括号里面的任何一个都能出数据，而mongodb中的$all则必须满足[]中的所有值。
1、{age:{$all:[7,9]}}:age数组中只要有7和9就满足条件。如果只有7，没有9则不符合条件。
Note:$exists判断字段是否存在，（true/false） 1、{city:{$exists:true}}: 集合中存在city这个字段的数据
Note：$mod取模运算。 1、{age:{$mod:[7,6]}}:集合中模7余6的数据 Note：$ne不等于 Note:$in包含，$nin不包含
。跟t-sql中的in，not in一样。 1、{age:{$in:[10,11]}}:如果age是数组的话，只要数组包含in中条件的任何一条数据，都能被检
索出来。不是数组，则只要满足in中的任何一个条件数据，也可以被检索出来。 Note：数组元素个数。
1、{age:{$size:4}}:age数组元素个数为4的数据结果集。 Note:$not正则匹配，不包含以张开头的数据。
Note:skip,limit,sort,count 1、skip（2）:从数据集的第二条开始查询 2、limit(2) : 依次查出2条数据。
3、sort({age:1}) : 1.正序查询，-1倒叙查询。 4、count():结果集总数。

