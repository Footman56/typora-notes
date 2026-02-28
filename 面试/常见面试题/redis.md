# 缓存穿透

缓存穿透是值 查询一个根本不存在的数据，数据不存在的话，redis里面没有，就直接查数据库。

1. 布隆过滤器 ： 快速判断key 是否存在
2. 将这个不存在的值存下来，value 为null



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