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

   ```java
   ```

   