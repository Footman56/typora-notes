# 缓存穿透

缓存穿透是值 查询一个根本不存在的数据，数据不存在的话，redis里面没有，就直接查数据库。

1. 布隆过滤器 ： 快速判断key 是否存在 (有一定的误判)

   1. 位数组 ，多个哈希函数

   ```
   m = 10000 位数组
   k = 3 个 hash 函数
   
   插入元素 "abc"：
   h1("abc") → 位置 5
   h2("abc") → 位置 200
   h3("abc") → 位置 888
   把这三个位置置为 1
   
   查询元素:查 "abc"：
   只要有一个位置为 0 → 一定不存在
   全部为 1 → 可能存在
   ```

2. 将这个不存在的值存下来，value 为null ，设置的TTL 时间短 300秒

3. 如果key 是连续的整数的话，可以使用Roaing Bitmap 来判断

   1. 分段：将32位整数拆成两部分，高16位为“桶索引”, 低16位 为 “桶内数据”
   2. 自适应
      + 桶内数据少时 ，使用Array COntainer  
      + 数据多时，使用Bitmap Container 

4. 数据需要频繁增删，布谷鸟过滤器

   ```
   1.计算两个候选桶位置
   2.若有空位 → 插入
   3.若都满 → 随机踢出一个元素
   4.被踢出的元素换到它的另一个桶
   5.可能形成“踢来踢去”
   ```

5. 立体防御
   1. 网关层：在nginx 网关层拦截恶意ip
   2. 应用层： 判断key不符合规则
   3. 数据层：缓存空对象或者过滤器

# 缓存击穿

单个热点key 过期，恰逢高并发过来，请求会访问数据库

1. 互斥锁 :  保证只有一个线程取查库

   ```java
   public Object getData（String key）｛
   // 1. 查询缓存
   Object value = redis.get （key）；
   if （value ！= nulL） return value；
   1/ 2. 缓存未命中，尝试获取互斥锁
   // SETNX key "1"EX 10（设置过期时间防止死锁）
   if（redis.tryLock（lockKey））｛
   try｛
   // Double Check：再次查询缓存，防止重复查库
   value = redis.get（key）；
   if （value ！= null） return value：
   // 3. 查询数据库
   value = db.query（key）；
   // 4. 回写缓存
   redis.set （key, value）；
   ｝ finally｛
   // 5.释放锁
   redis.unlock（ LockKey）；
   ｝
   ｝ else｛
   // 6.获取锁失败，休眠后重试
   Thread.sleep（50）；
   return getData（key）：
   ｝
   return
   value；
   ```

2.  逻辑过期 （key 永不过期，value 中存在过期时间戳，后台线程异步重建）



# 缓存雪崩

大面积key 同一时刻失败

1. TTL 随机化， 在基础时间上加一个随机值
2. 架构层面：
   1. Redis  Sentinel (哨兵) ：监控主节点状态，自动完成故障转移
   2. Redis Cluster (集群)： 通过分片存储数据，不至于全盘皆输





# redis 持久化

## RDB: 内存快照

默认开启为RDB持久化

在指定时间间隔内将内存中的数据写入到磁盘。

1. fork 子进程备份
2. 将共享内存数据写入临时RDB文件（COW 写时复制）
3. 完成临时写入，替换旧的RDB 文件，退出

### 触发时机

1. 自动触发

   ```
   save 900 1
   save 300 10
   save 60 10000
   
   900秒内有1次修改
   300秒内有10次修改
   60秒内有10000次修改就生成一次快照。
   ```

2. 手动触发

   ```
   SAVE  # 同步阻塞
   BGSAVE ¥ 异步阻塞
   ```

3. shutdown 触发  
4. flushall 触发

优点：

文件小、恢复快、适合做冷备份、性能影响小

缺点：

不是实时持久化、两次快照之间数据会丢

## 增量日志（AOF）

把每条写命令追加到AOF缓冲区中，根据同步策略写到AOF文件，当AOF文件达到重写策略配置的阈值时，会重写，AOF瘦身, redis 重启时 重新执行AOF 文件里面命令   bgrewirteao可以重写AOF文件

默认不开启  appendonly yes

### 刷盘策略

```
# 每次命令都fsync 最安全、性能最差
appendfsync always  

# 每秒fsync 一次 平衡 最多丢1秒数据(默认)
appendfsync everysec

# 交给os 决定，丢很多数据，性能最好
appendfsync no
```

### 文件碰涨

重写是对AOF重复性指令的整理。

 auto-aof-rewrite-percentage 100：当AOF文件体积达到上次重写之后的体积的100%时，会
触发AOF重写。
• auto-aof-rewrite-min-size 64mb：当AOF文件体积超过这个阈值时，会触发AOF重写。
当AOF文件的体积达到或超过上次重写之后的比例，并且超过了最小体积阈值时，Redis会自动触
发AOF重写新的AOF文件  
优点： 数据更加可靠、可以保留写命令历史

缺点：文件比较大、恢复速度慢

## 混合持久

redis 4.0 支持混合持久化 ，默认开启

# 过期策略

## 惰性删除

当客户尝试访问某个健时，Redis 会先检查是否过期，过期就删除。

## 定期删除

Redis 每隔一段时间（100毫米） 随机检查一部分过期时间的健，会随机从16个库中 抽取一定数量的key进行判断是否过期. 当抽查的缓存中,超过25%的key过期,那么会再次抽查,直到小于25%

同时采用定期删除和惰性删除。  还是会有大量的内存中key是已经失效的，导致内存溢出问题，解决方式引入淘汰机制

# 内存淘汰机制

## 不淘汰策略

新写入的命令返回错误，写操作失败

## volatile-lru (最近最少使用)

优先删除最久未被访问的键，保留常用的键

## volatile-tel( 根据过期时间优先)

优先删除剩余时间较短的键，保留剩余时间更长的

## volatile-random

随机删除一个键

## alleys-lru(全局最近最少使用)

从所有键中选择最少使用的键进行删除。无论键是否设置了过期时间，都将参与淘汰。

## allkeys-random（全局随机删除）

