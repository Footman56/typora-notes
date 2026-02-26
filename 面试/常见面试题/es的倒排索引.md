倒排索引就是将这些词汇映射到包含这些词汇的文档列表中的。

# 倒排索引构成

1. 单词词典
   + 记录所有不重复的单词
   + 通常使用B+树或者哈希表来实现
   + 快速定位单词的倒排列表
2. 倒排列表
   + 记录包含改单词的所有文档ID
   + 包括词频、位置信息

# es 构建过程

## 步骤1：文本分析（Analysis）

```json
原始文档：{"title": "Elasticsearch is awesome"}

分词过程：
1. 字符过滤：去除HTML标签等
2. 分词器：分成单词
   → ["Elasticsearch", "is", "awesome"]
3. Token过滤：转小写、去除停用词
   → ["elasticsearch", "awesome"]
```

## 步骤2：建立索引

```json
文档1：elasticsearch, awesome
文档2：elasticsearch, java
文档3：awesome, programming

生成的倒排索引：
elasticsearch → [1,2]
awesome       → [1,3]
java          → [2]
programming   → [3]
```

## 步骤3：写入磁盘

+ 生成Segment文件
+ 包括倒排索引和存储字段



# 倒排索引结构优化

1. FOR 压缩

   ```java
   // 未压缩的文档ID列表
   [1, 3, 4, 7, 10, 1000, 1001, 1002]
   
   // 1. 先排序（已经是排序的）
   // 2. 计算差值（Delta Encoding）
   [1, 2, 1, 3, 3, 990, 1, 1]  // 相邻差值
   
   // 3. 按块压缩
   块1: [1,2,1,3,3] → 最大3，用2bit存储
   块2: [990,1,1]   → 最大990，用10bit存储
   
   // 节省大量空间
   ```

2.  Roaring Bitmaps（位图优化）
3. 跳表 

elastic search 使用倒排索引和正排索引：

倒排索引用于搜索。

正排索引用于排序和聚合

# es 查询为什么那么快？

1. 倒排索引  能直接找到关键词的页面 ，从O(n)降到O(1)

2. 不可变Segment + 缓存 ： 写入页面后不可变、热门segment常驻缓存、充分利用系统的Page Cache

3. 压缩算法 减少IO   FOR压缩、Roaring Bitmaps

4. 分布式并行查询

   ```java
   // 1. 协调节点接收请求
   // 2. 计算目标分片（默认所有分片）
   // 3. 并行发送请求到各分片
   // 4. 每个分片本地查询返回top N
   // 5. 协调节点归并排序
   ```

5. 字段数据缓存
   + 节点级别缓存：所有分片共享
   + LRU 淘汰： 最近最少使用
   + 内存优化，使用堆外内存
6. 跳表加速 ： 有跳表层
7. 索引分片与路由     // 指定routing值，直接定位分片

# es 数据多了怎么调优

1. 控制内存大小不能超过 32G(java 使用指针压缩技术，将64位压缩到32位)   保留 50%内存给文件系统缓存

2. 冷热分离 与分片策略   单个分片大小最好在 30～50GB. ILM 实现

   1. 给节点打角色标签
   2. 定义生命周期策略 （多少天为冷？多少天为热）

   本质：冷热分离 = 用 ILM 把不同时间段的索引迁移到不同性能节点，减少热节点的 shard 数量，从而提升查询性能并降低成本。

    业务搜索冷热不是由时间决定的

   1. 将索引拆成冷、热两个索引
   2. 应用层实现 不让冷数据参与搜索
      + redis 缓存热门结果
      + 热数据预计算
      + 冷数据 只在深度翻页时查询

   使用Rollover API ，当索引 Size > 50GB 或者Time > 7days 时自动滚动

   ```java
   Rollover 依赖一个 写别名（write alias）。
   1. 创建生命周期策略
   PUT _ilm/policy/my_policy
   {
     "policy": {
       "phases": {
         "hot": {
           "actions": {
             "rollover": {
               "max_size": "30gb",
               "max_age": "7d"
             }
           }
         }
       }
     }
   }
   
   2. 创建索引模板
   PUT _index_template/my_template
   {
     "index_patterns": ["my_index-*"],
     "template": {
       "settings": {
         "index.lifecycle.name": "my_policy",
         "index.lifecycle.rollover_alias": "my_index_write",
         "number_of_shards": 3
       }
     }
   }
   3.  创建第一个索引
   PUT my_index-000001
   {
     "aliases": {
       "my_index_write": {
         "is_write_index": true
       }
     }
   }
   4. 创建别名
   POST _aliases
   {
     "actions": [
       {
         "add": {
           "index": "my_index-*",
           "alias": "my_index_read"
         }
       }
     ]
   }
   5.查询
    GET my_index_read/_search
   {
     "query": {
       "match": {
         "name": "iphone"
       }
     }
   }
   ```

3. 海量写入 CPU 飙升

   + 使用Bulk API ，每次5～15MB
   + 将