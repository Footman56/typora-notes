# 一、数据预热

通过后台任务 在低峰期间 定期访问热点数据

1. 识别热点数据，根据业务情景、时间

2. 构建预热查询 ：通常bool 结合range ， 将size=0，只加载数据
3. 调度

# 二、冷热数据分离

ILM 实现

1.定义节点角色：在不同节点的 elasticsearch.yml中设置属性，区分热节点和冷节点。

```
热节点（高性能 SSD）

node.attr.box_type: hot

 冷节点（大容量 HDD）

node.attr.box_type: cold
```



2. 创建ILM 策略：定义一个策略，规定数据在不同阶段（hot, warm, cold. delete）的行为。

```

''policy"： ｛
"'phases"：｛
"hot"：｛
"actions"：
｛
"rollover"：｛ "max_age"："7d"｝
｝
"cold"：｛
'min_age"： "30d"，
''actions"： ｛
"allocate"： ｛ "require"： ｛ "box_type"： "cold" ｝ ｝
｝
｝
｝
｝
｝
```

3. 关联索引模版

   在索引模版中指定ILM策略 和热数据节点分配规则，

# 三、字段精简和预处理

在索引中适当的冗余多个字段，避免查询多个e s做数据join

# 四、分页优化

1. form +size
2. scroller  
3. search_after  : 基于上页的排序结构