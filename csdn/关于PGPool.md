   pgpool-II 是一个位于 PostgreSQL 服务器和 PostgreSQL 数据库客户端之间的中间件，它提供以下功能： 连接池 复制
负载均衡 并行查询 HA 查询缓存 功能可以说非常丰富，但实际的效果如何还要看和场景的是否契合。 进程架构
PGPool和PostgreSQL一样也采用了多进程架构，应该是受PG影响，连很多代码都差不多。PGPool启动后会包含以下进程
主进程(PgpoolMain) 对所有backend实施健康检查，如果有后端PostgreSQL宕机，发起failover
watchdog(watchdog_main) 多个PGPool间的通信，配置同步等，没有细看。 lifecheck(lifecheck_main)
watchdog的另一个进程，检查上流信任的服务器连接(trusted_servers)，检查收到的其它节点的心跳 child进程(do_child)
child进程数为numinitchildren，默认32。numinitchildren代表了最大并发数，超过客户端将阻塞等待。
每个child进程循环执行下面的任务 1)select()检查前端socket 2)accept()接收客户端连接
3)get_connection()根据客户端连接socket创建前端连接对象 4)get_backend_connection()
4.1)connect_backend() 新建后端连接，一个后端连接对象实际包含了到每个后端的socket，
保存在POOL_CONNECTION_POOL->slots数组里。新建的连接需要放到连接池，如果连接池已满，
淘汰一个最老的连接对象。连接池的大小为max_pool，默认为4。 或
4.2)connect_using_existing_connection()从连接池获取后端连接(需要连接的db，用户名等相同)。
循环执行pool_process_query()，将前端的请求转发到合适的后端。 PCP进程(pcp_main) 接收并处理PCP请求
work进程(doworkerchild) 1)检查并执行reload_config请求 2)检查并执行restart_request请求
3)检查streaming replication mode下的主备延迟 功能说明 连接池 pgpool-II 保持已经连接到 PostgreSQL
服务器的连接，并在使用相同参数（例如：用户名，数据库，协议版本） 连接进来时重用它们。它减少了连接开销，并增加了系统的总体吞吐量。 参考PGPool连接池的
实现，它虽然通过缓存物理连接减少了连接开销，但只做到了会话级的连接池，并不具有缓解高并发压力的功能。考虑到cancel需要额外的连接，反而大大降低了系统可承
受的连接数上限。对应用提供最大连接数只有数据库承受的最大连接数的1/8。之所以这样时有PGPool的多进程架构决定了，一个子进程对应于一个应用连接，每个子进
程占用max_pool个后端连接，考虑到取消查询的需求，那又要双倍的后端连接。所以对高并发业务使用PGPool一定要小心，应用端不能有长连接，即不能部署连接
池，业务每次使用数据库连接的时间尽可能短，用完立刻释放。 归纳起来，max_pool，num_init_children，max_connections 和
superuser_reserved_connections 必须符合以下规则： max_pool*num_init_children <=
(max_connections - superuser_reserved_connections) （不需要取消查询）
max_pool*num_init_children*2 <= (max_connections -
superuser_reserved_connections) （需要取消查询） 复制 pgpool-II 可以管理多个 PostgreSQL
服务器。激活复制功能并使在2台或者更多 PostgreSQL 节点中建立一个实时备份成为可能， 这样，如果其中一台节点失效，服务可以不被中断继续运行。
从功能上复制应该包含replication_mode和master_slave_mode 2种情况
replication_mode的实现方式就是分发SQL到多个节点，达到冗余高可用的目的，对SELECT语句可以设置是复制还是只发往其中一个节点。
基于SQL分发的复制容易出现主备数据不一致的问题，在有PG原生流复制可以用的情况下，HA没必要使用复制模式。
master_slave_mode=true且master_slave_sub mode='stream'时采用流复制模式，推荐采用这一模式做HA。
负载均衡 如果数据库进行了复制，则在任何一台服务器中执行一个 SELECT 查询将返回相同的结果。pgpool-II 利用了复制的功能以降低每台
PostgreSQL 服务器的负载。它通过分发 SELECT 查询到所有可用的服务器中，增强了系统的整体吞吐量。在理想的情况下， 读性能应该和
PostgreSQL 服务器的数量成正比。负载均很功能在有大量用户同时执行很多只读查询的场景中工作的效果最好。
负载均衡或准确的说是读负载均衡，根据一些已知的必须发给Master的特征SQL，和可配置函数白名单黑名单决定把SQL发给Master还是Standby。
限制超过限度的连接 PostgreSQL 会限制当前的最大连接数，当到达这个数量时，新的连接将被拒绝。增加这个最大连接数会增加资源消耗并且对系统
的全局性能有一定的负面影响。pgpoo-II 也支持限制最大连接数，但它的做法是将连接放入队列，而不是立即返回一个错误。 并行查询
使用并行查询时，数据可以被分割到多台服务器上，所以一个查询可以在多台服务器上同时执行，以减少总体执行时间。 并行查询在查询大规模数据的时候非常有效。 这其实
就是sharding，并行查询必须和replication_mode以及负载均衡同时使用，通过系统表定义sharding表和sharing键，sharing
表以外的是复制表。 由于并行模式限制比较多，用户很少，3.5中已经将并行查询删除了。 查询缓存 查询缓存可以在 pgpool-II
的所有模式中使用，内部通过memcached实现。 小结 PGPool功能丰富，但很多和类似产品比起来似乎不够好。连接池没有pgbouncer好，shard
ing也不如citus，XC/XL/X2。基于流复制的HA+读写分离+读负载均衡也许是个不错的应用场景。 参考
http://pgpool.projects.pgfoundry.org/pgpool-II/doc/pgpool-zh_cn.html
http://www.pgpool.net/docs/latest/pgpool-en.html https://www.sraoss.co.jp/even
t_seminar/2016/edb_summit_2016.pdf#search='postgresql+HA'

