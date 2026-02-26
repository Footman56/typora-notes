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

