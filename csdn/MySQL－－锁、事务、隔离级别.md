为什么需要锁？因为数据库要解决并发控制问题。在同一时刻，可能会有多个客户端对表中同一行记录进行操作，比如有的在读取该行数据，其他的尝试去删除它。为了保证数据
的一致性，数据库就要对这种并发操作进行控制，因此就有了锁的概念。

