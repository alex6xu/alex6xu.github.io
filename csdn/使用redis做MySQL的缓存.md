1.脚本同步： 自己写脚本将数据库数据写入到redis/memcached。 这就涉及到实时数据变更的问题（mysql row
binlog的实时分析），binlog增量订阅Alibaba 的canal ，以及缓存层数据 丢失/失效 后的数据同步恢复问题。 2.业务层实现：
先读取nosql缓存层，没有数据再读取mysql层，并写入数据到nosql。
nosql层做好多节点分布式（一致性hash），以及节点失效后替代方案（多层hash寻找相邻替代节点），和数据震荡恢复了。

