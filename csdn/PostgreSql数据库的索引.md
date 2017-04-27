PostgreSql数据库的索引分为B-tree, Hash, GiST,SP-GiST and GIN。这边先只讨论b-
tree索引。B-tree是最常用的的索引，并且PG在创建索引时如果没有明确的指定索引的类型那么默认就是b-tree(B树索引)索引,如：
Create index index_nameon table_name(column_name,[…]);
B-tree索引支持等值和值范围的查询，这些体现在子查询中，主要支持的操作符是： =,<,>,<=,>=。 除了以上的操作符支持b-
tree索引外，between .. and .. ，以及is null,is not null。 Is null确实支持b-tree索引，如创建一个表：
Create table t3(a varchar(10), bvarchar(10), c varchar(10)); insert into t3
select generate_series(1,10000)||'a',generate_series(1,10000)||'b',
generate_series(1,10000)||'c'; select
generate_series(1,10000)||'a',generate_series(1,10000)||'b',
generate_series(1,10000)||'c'; 创建索引： create index idx_t3_2 on t3(col1)；
explain select * from t3 where col1 is null; "QUERYPLAN" "Index Scan using
idx_t3_2on t3  (cost=0.29..8.30 rows=1width=15)" "  Index Cond: (col1 IS
NULL)"; 但是当使用is not null 时，执行的结果是： "QUERYPLAN" "Seq Scan on t3
(cost=0.00..163.01 rows=10000 width=15)" "  Filter: (col1 IS NOT NULL)"
并没有使用索引idx_t3_2。 通过上面的描述B-tree索引的使用时在一定的查询条件下才能使用，比如需要等值条件(“=”)，值的范围(“>=”)，is
null，那么当使用<>,!=,以及or等符号时就不会使用索引了。 上面讨论的是单字段的索引，当多字段时存在不一样情况。 create index
idx_t3_1 on t3(col1,col2,col3); 如果需要使用索引idx_t3_1，对于SQL语句的编写有特殊的要求：
1、索引中的第一个字段需要时等值。这一点是非常重要的，不然是不会使用该索引的； 2、其他字段可以是等值也可以是范围；
3、多字段索引的字段个数是有限的，默认下是32个字段，如果需要改变是可以在PostgreSql安装时修改pg_config_manual.h。 例如：
create index idx_t3_1 ont3(col1,col2,col3); explain select * from t3
wherecol1=’1’ and col2=’1’ and col3=’1’;     "QUERY PLAN" "Index Only Scan
usingidx_t3_1 on t3  (cost=0.29..8.31 rows=1width=15)" "  Index Cond: ((col1 =
'1'::text) AND (col2 ='1'::text) AND (col3 = '1'::text))"   如果SQl是这样就不会使用索引：
Explian select * from t3 wherecol1>’1’ and col2=’1’ and col3=’1’;   "QUERY
PLAN" "Seq Scan on t3  (cost=0.00..238.02 rows=1 width=15)" "  Filter:
(((col1)::text > '1'::text) AND((col2)::text = '1'::text) AND ((col3)::text =
'1'::text))"

