全局锁：锁住整个实例，只能读，备份时使用，后面mysql备份时不需要加全局锁 （MVCC 实现无锁备份） 全局锁适合MYisam等不支持事务的引擎

表锁 ：锁定整张表   开销小，加锁快，并发度低

+ 表级读锁：多个事务可以同时加读锁，读读不互斥，读写互斥
+ 表级写锁：仅一个事务可以加写锁，写写，写读互斥
+ MDL锁： 执行增删改查时自动加MDL 读锁，执行ALTER TABLE 时自动加MDL 写锁  写写，写读互斥

长事务导致DML写锁阻塞，后续对表的修改都会排队阻塞

show  processlist  查看进程状态



行级锁 （锁单行） 仅在事务中生效

生产坑点：行级锁依赖索引，如果索引失效时，全表扫描时，等同于表锁

共享锁（S锁） 读读兼容，读写锁定  SELECT ... LOCK IN SHARE MODE

排他锁（X锁） SELECT  ... FOR UPDATE  或者insert、del、 update  



临建锁是innodb的默认行锁算法，可重复读隔离级别

唯一索引的等值查询： 临建锁降为记录锁

唯一索引的范围查询：仍为临建锁（锁定范围 + 间隙）  左开右闭

非唯一索引的等值/范围查询 ：仍为临建锁



超卖问题：在未提交读这种隔离级别下，无锁，多个事务更新，导致计数错误

+ 修改未提及读
+ 通过X 锁定目标行



通过show engine innodb status 来分析死锁

死锁解决方案：

1. 所有事务按照相同的顺序锁定行
2. 减少锁持有时间
3. 设置锁超时



意向锁（快速判断表中是否有行级锁）：

如果事务准备对行进行加s锁，会自动在表级上 IS 锁， 如果在行准备加X 锁，会自动在表级上 IX 锁。意向锁紧与表级锁互斥



锁相关排查命令

```
show processlist

show full processlist


-- 重要提示：MySQL 8.② 新特性（旧表innodb_locks已移除）
-- 1. 查看当前所有锁信息（替代原innodb_locks）
SELECT * FROM performance_schema.data_locks；

-- 2. 查有锁等待关系（替代原innodb_lock_waits）
-- 核心字段：blocking_engine_transaction_id（阻塞事务ID）、requesting_engine_transaction_id（请求事务ID）
SELECT
blocking_engine_transaction_id AS blocking_trx_id，
-- 阻塞方事务ID
equesting_engine_transaction_id AS requesting_trx_id，-- 请求方事务ID
blocking_lock_id， -- 阻塞方锁ID
requesting_lock_id -- 请求方锁ID
FROM performance_schema.data_lock_waits：


-- 3. 结合进程表，定位阻塞源对应的SQL（适配8.0新表）
SELECT
p.id，-- 进程ID
p.user，
p.host，
p.db，
p.info， -- 执行的SQL
L.lock_type， -- 锁类型（TABLE/ROW）
L.lock_mode，
-- 锁模式（X/S/IS/IX等）
L.lock_table -- 锁定的表
FROM
information_schema.processlist p
JOIN
WHERE
performance_schema.data_locks 1 ON p.id = SUBSTRING_INDEX（L. lock_id，'：'，1）
p.state LIKE 'Waiting for%'；-- 筛选等待锁的进程
```

