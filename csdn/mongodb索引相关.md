索引是提高查询查询效率最有效的手段。索引是一种特殊的数据结构，索引以易于遍历的形式存储了数据的部分内容（如：一个特定的字段或一组字段值），索引会按一定规则对
存储值进行排序，而且索引的存储位置在内存中，所在从索引中检索数据会非常快。如果没有索引，MongoDB必须扫描集合中的每一个文档，这种扫描的效率非常低，尤其
是在数据量较大时。 创建索引查看索引删除索引 1\. 创建索引ensureIndex() MongoDB创建索引使用ensureIndex()方法。
语法结构 db.COLLECTION_NAME.ensureIndex(keys[,options])
keys，要建立索引的参数列表。如：{KEY:1}，其中key表示字段名，1表示升序排序，也可使用使用数字-
1降序。options，可选参数，表示建立索引的设置。可选值如下：
background，Boolean，在后台建立索引，以便建立索引时不阻止其他数据库活动。默认值
false。unique，Boolean，创建唯一索引。默认值 false。name，String，指定索引的名称。如果未指定，MongoDB会生成一个索引
字段的名称和排序顺序串联。dropDups，Boolean，创建唯一索引时，如果出现重复删除后续出现的相同索引，只保留第一个。sparse，Boolean，
对文档中不存在的字段数据不启用索引。默认值是 false。v，index version，索引的版本号。weights，document，索引权重值，数值在
1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 如，为集合sites建立索引： >
db.sites.ensureIndex({name: 1, domain: -1}) { "createdCollectionAutomatically"
: false, "numIndexesBefore" : 1, "numIndexesAfter" : 2, "ok" : 1 } 2\. 查看索引 Mo
ngoDB提供了查看索引信息的方法：getIndexes()方法可以用来查看集合的所有索引，totalIndexSize()查看集合索引的总大小，reInd
ex()查看集合的索引信息。 2.1 查看集合中的索引getIndexes() db.COLLECTION_NAME.getIndexes()
如，查看集合sites中的索引： >db.sites.getIndexes() [ { "v" : 1, "key" : { "_id" : 1 },
"name" : "_id_", "ns" : "newDB.sites" }, { "v" : 1, "key" : { "name" : 1,
"domain" : -1 }, "name" : "name_1_domain_-1", "ns" : "newDB.sites" } ] 2.2
查看集合中的索引大小totalIndexSize() db.COLLECTION_NAME.totalIndexSize()
如，查看集合sites索引大小： > db.sites.totalIndexSize() 16352 2.3 查看集合中的索引信息reIndex()
db.COLLECTION_NAME.reIndex() 如，查看集合sites的索引信息： > db.sites.reIndex() {
"nIndexesWas" : 2, "nIndexes" : 2, "indexes" : [ { "key" : { "_id" : 1 },
"name" : "_id_", "ns" : "newDB.sites" }, { "key" : { "name" : 1, "domain" : -1
}, "name" : "name_1_domain_-1", "ns" : "newDB.sites" } ], "ok" : 1 } 3\. 删除索引
不在需要的索引，我们可以将其删除。删除索引时，可以删除集合中的某一索引，可以删除全部索引。 3.1 删除指定的索引dropIndex()
db.COLLECTION_NAME.dropIndex("INDEX-NAME")
如，删除集合sites中名为"name_1_domain_-1"的索引： > db.sites.dropIndex("name_1_domain_-1")
{ "nIndexesWas" : 2, "ok" : 1 } 3.3 删除所有索引dropIndexes()
db.COLLECTION_NAME.dropIndexes() 如，删除集合sites中所有的索引： > db.sites.dropIndexes() {
"nIndexesWas" : 1, "msg" : "non-_id indexes dropped for collection", "ok" : 1
}

