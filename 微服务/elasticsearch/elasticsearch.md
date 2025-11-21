elasticsearch-head

elastic search-head 是谷歌插件，可以查看elasticsearch 存储情况

直接在谷歌商店下载即可



##  添加文档

POST http://localhost:9200/索引/__doc  或者  http://localhost:9200/shopping3/_create

_doc 是类型，现在都是 _doc这一种类型 

```
POST http://localhost:9200/shopping3/_doc
body 中添加json格式的内容
{
    "name":"xiaoli",
    "age":12,
    "sex":"girl"
}
```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220719231536540.png" alt="image-20220719231536540" style="zoom:50%;" />

多次发送请求会创建多个文档

POST http://localhost:9200/索引/_doc/指定Id

```
http://localhost:9200/shopping3/_doc/100
```

数据存储：

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220719231742574.png" alt="image-20220719231742574" style="zoom:50%;" />

## 查询

### 主键查询

GET http://localhost:9200/索引/_doc/id主键

```
GET http://localhost:9200/shopping3/_doc/100
```

```json
{
    "_index": "shopping3",
    "_type": "_doc",
    "_id": "100",
    "_version": 1,
    "_seq_no": 3,
    "_primary_term": 1,
  // 是否找到
    "found": true,
    "_source": {
        "name": "xiaoli",
        "age": 12,
        "sex": "girl"
    }
}
```

### 全查询

GET http://localhost:9200/索引/_search

```
http://localhost:9200/shopping3/_search
```

```json
{
  // 耗时，毫秒单位
    "took": 2,
  //  是否超时
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": 1.0,
      // 命中
        "hits": [
            {
                "_index": "shopping3",
                "_type": "_doc",
                "_id": "VwwEF4IB0bzTGXVT6bpN",
                "_score": 1.0,
                "_source": {
                    "name": "xiaoli",
                    "age": 12,
                    "sex": "girl"
                }
            },
            {
                "_index": "shopping3",
                "_type": "_doc",
                "_id": "WAwHF4IB0bzTGXVTvLpf",
                "_score": 1.0,
                "_source": {
                    "name": "xiaoli",
                    "age": 12,
                    "sex": "girl"
                }
            },
            {
                "_index": "shopping3",
                "_type": "_doc",
                "_id": "WQwHF4IB0bzTGXVTwLo2",
                "_score": 1.0,
                "_source": {
                    "name": "xiaoli",
                    "age": 12,
                    "sex": "girl"
                }
            },
            {
                "_index": "shopping3",
                "_type": "_doc",
                "_id": "100",
                "_score": 1.0,
                "_source": {
                    "name": "xiaoli",
                    "age": 12,
                    "sex": "girl"
                }
            },
            {
                "_index": "shopping3",
                "_type": "_doc",
                "_id": "200",
                "_score": 1.0,
                "_source": {
                    "name": "xiaoli",
                    "age": 12,
                    "sex": "girl"
                }
            }
        ]
    }
}
```

### 简单条件查询

GET http://localhost:9200/shopping3/_search?q=name:xiaoli  直接在请求路径中添加查询条件

也可以组装多个条件

http://localhost:9200/megacorp/_search?q=last_name:liu&q=first_name:xiaoli

GET 

Body

```json
{
  // 查询条件
    "query":{
  // 查询全部
        "match_all":{
        }
  //条件查询
        "match":{
          "name":"lizhi"
        }
    },
   // 从哪里开始 
    "from" :0,
   // 每页数据
    "size":3,
   // 返回结果列
    "_source":["name","sex"],
   // 排序
    "sort":{
        "age":{
            "order":"asc"
        }
    }
}
```

[match] query doesn't support multiple fields, found [name] and [sex]： match不支持多个条件

[match_all] unknown field [name] did you mean [_name]? : match_all 不支持条件

### 多条件查询

一些格式是固定的

```json
{
  "query":{
    "bool":{
      "must":{
        "macth":{
          "xxxx":"xxxxx"
        }
      },
      "filter":{
        "range":{
          "xx":{"gt":xxx}
        }
      }
    }
  }
}
```



must = and ,should = or  filter 过滤

```json
{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                         "name": "xiaoli"
                    }
                },{
                    "match":{
                        "age":13
                    }
                }
            ]
        }
    }
}
```

```json
{
    "query":{
        "bool":{
            "should":[
                {
                    "match":{
                         "name": "xiaoli"
                    }
                },{
                    "match":{
                        "name":"lizhi"
                    }
                }
            ]
        }
    }
}
```

```json
{
    "query":{
        "bool":{
            "should":[
                {
                    "match":{
                         "name": "xiaoli"
                    }
                },{
                    "match":{
                        "name":"lizhi"
                    }
                }
            ],
            "filter":{
                "range":{
                    "age":{
                        "gt": 12
                    }
                }
            }
        }
    }
}
```

使用match 就是全文检索。elaticsearch 将文档建立倒排索引，match 在查询的时候会分成一个词去查询索引，

使用match_phrase 就是精确查询

## 全量修改

PUT http://localhost:9200/索引/_doc/id

body 里面放修改内容json

```
PUT http://localhost:9200/shopping3/_doc/100
 {
                    "name": "xiaolixxxx",
                    "age": 15,
                    "sex": "boy"
}
```

```json
{
    "_index": "shopping3",
    "_type": "_doc",
    // 影响的主键
    "_id": "100",
  //  每做一次操作 +1
    "_version": 3,
  //  更新类型
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 6,
    "_primary_term": 1
}
```

多次执行时都会更新

## 部分修改

POST http://localhost:9200/索引/**_update**/id

BODY 

```json
{
    "doc":{
      // 要修改的内容
        "name":"lizhi"
    }
}
```

一定要指定为_update ，否则就是全文替换

```json
{
    "_index": "shopping3",
    "_type": "_doc",
    "_id": "200",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 9,
    "_primary_term": 1
}
```



## 删除内容

DELET  http://localhost:9200/索引/_doc/id

## 聚合操作

GET http://localhost:9200/shopping3/_search

Body

```json
{
    // 聚合操作
    "aggs":{
        // 分组名称
        "group_name":{
            // 分组操作
          // avg 求平均值
            "terms":{
                // 分组字段
                "field": "name"

            }
        }
    },
  // 不展示原数据，只展示统计数据
    "size" :0
}
```

如果字段没有进行优化，也类似没有加索引。没有优化的字段es默认是禁止聚合/排序操作的。

加索引操作

PUT http://localhost:9200/索引/_mapping?pretty

body

```json
{
    "properties": {
        // 要优化的字段
    "name": { 
      "type":     "text",
      "fielddata": true
    }
  }
}
```

## 映射

映射：可以理解成定义对象

先建立索引

再创建参数映射

Body：

PUT http://localhost:9200/索引/_mapping

```json
{
    "properties":{
        "name":{
          // 可分词，全文索引
            "type": "text",
            "index": true
        },
        "sex":{
          // 不能分词，完全匹配
            "type":"keyword",
            "index":true
        },
        "tel":{
            "type":"text",
            "index":false
        }
    }
}
```



注在5.x 版本中 如果数值型字段不需要做范围查询的话，就设置成keyword，这样查询性能快

[]:

