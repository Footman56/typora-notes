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
   ```

   



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