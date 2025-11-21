# 索引相关

索引必须是小写，不能以“_”开始

## 创建索引

```
# 创建huochai的索引
PUT /huochai

# 创建huochai的索引指定分片3 个、副本数量2个
PUT /huochai{
  "settings":{
  	"number_of_shards": 3,
  	"number_of_replicas": 2
  }
}
```

## 修改索引

可以修复副本数量，但是不能修改分片数量

```
PUT /huochai/_settings{
   "number_of_replicas": 4
}
```

## 查询索引

```
GET /huochai

# 检查索引是否存在
HEAD /huochai
```

## 删除索引

```
DELETE /huochai
```

# 文档操作

## 添加数据

 [PUT | POST] /索引名称/[_doc | _create ]/id

```
# PUT 添加的数据的时候必须要指定id，否则会失败
# 如果文档中没有指定id 的话就会创建一个id,有的话就会删除现有文档，在新增文档，版本号+1
PUT /huochai/_doc/1
{
"name":"张三",
"sex":1,
"age":25,
"address":"广州天河公园",
"remark":"javadeveloper"
}
```

```
# POST 没有指定id 的话，es 会自动生成一个id,新生成的文档版本号是1
# 并且多次执行的话，会生成多个文档，id不同，版本号为1
POST /huochai/_doc
{
"name":"张三",
"sex":1,
"age":25,
"address":"广州天河公园",
"remark":"javadeveloper"
}
```

```
# 如果指定了id ,每次执行完就是更新文档操作，版本号就+1
POST /huochai/_doc/Za_eJIYBVRuPqXZvBgVc
{
"name":"张三",
"sex":1,
"age":25,
"address":"广州天河公园",
"remark":"javadeveloper"
}
```

```
# 创建文档，如果文档id 已经存在的话，就创建失败
POST /huochai/_doc/Za_eJIYBVRuPqXZvBgVc
{
"name":"张三",
"sex":1,
"age":25,
"address":"广州天河公园",
"remark":"javadeveloper"
}
```

## 修改文档

[PUT | POST] /索引名称/_doc/id

```
# 指定id的话，文档还是存在的，内容会更新，版本会+1
# 全量替换
PUT /huochai/_doc/1
{
  "name":"张三1",
  "sex":1,
  "age":25,
  "address":"广州天河公园",
  "remark":"javadeveloper"
}


# 全量替换
POST /huochai/_doc/1
{
  "name":"张三1",
  "address":"广州天河公园",
  "remark":"javadeveloper"
}
```

## 部分更新

POST /索引名称/_update/id

```
# 只更新name 字段内容，做不到删除某个字段
POST /huochai/_update/1
{
  "doc": {
    "name": "李四"
  }
}
```

## 查询更新

通过 _update_by_query 

POST /索引名称/_update_by_query 

```
POST /huochai/_update_by_query
{
  # 先查询数据
	"query":{
		"match":{
			"_id": 1
		}
	},
	# 再执行更新，直接通过脚步修改文档的字段
	"script":{
		"source": "ctx._source.name = '王五' "
	}
}
```

## 并发修改

并发修改是通过版本控制来保证修改的一致性。在更新的时候执行版本，如果版本不一致的话就更新失败

```
POST /huochai/_doc/1?if_seq_no=10&if_primary_term=1
{
	"doc":{
		"name": "赵六" 
	}
}
```

![image-20230206143630485](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302061436552.png)

## 查询

ES Search API提供了两种条件查询搜索方式: 

+ REST风格的请求URI，直接将参数带过去，参数在请求url中

+ 封装到request body中，这种方式可以定义更加易读的JSON格式

```
#通过URI搜索，使用“q”指定查询字符串，“querystringsyntax”KV键值对 2
#条件查询, 如要查询age等于28岁的 _search?q=*:***
GET /es_db/_doc/_search?q=age:28

#范围查询, 如要查询age在25至26岁之间的 _search?q=***[** TO **] 注意: TO 必 须为大写
GET /es_db/_doc/_search?q=age[25 TO 26]

#查询年龄小于等于28岁的 :<= 
GET /es_db/_doc/_search?q=age:<=28 
#查询年龄大于28前的 :>
GET/es_db/_doc/_search?q=age:>28

#分页查询 from=*&size=* 
GET /es_db/_doc/_search?q=age[25 TO 26]&from=0&size=1

#对查询结果只输出某些字段 _source=字段,字段
GET /es_db/_doc/_search?_source=name,age
#对查询结果排序 sort=字段:desc/asc 
GET /es_db/_doc/_search?sort=age:desc
```



## 删除文档

DELETE  /索引名称/_doc/id



# 批量操作

批量操作可以分为：批量创建、批量插入/更新 、批量删除、 批量部分更新、批量读取

**这些操作都是通过对_id 来处理的**

批量对文档进行写操作是通过_bulk的API来实现的
 请求方式:POST 请求参数:通过_bulk操作文档，一般至少有两行参数(或偶数行参数) 请求地址:_bulk

参数类似于:

第一行参数为指定操作的类型及操作的对象(index,type和id)

第二行参数才是操作的数据

"_index": 要操作的索引

"_type"： 数据类型

"_id"： 要操作的文档

```
{"actionName":{"_index":"indexName","_type":"typeName","_id":"id"}} 
{"field1":"value1","field2":"value2"}

# actionName:表示操作类型，主要有create,index,delete和update
```



## 批量创建

actionName = create

```
POST _bulk
 {"create":{"_index":"article","_type":"_doc","_id":3}}
 {"id":3,"title":"fox老师","content":"fox老师666","tags":["java","面向对 象"],"create_time":1554015482530}
 {"create":{"_index":"article","_type":"_doc","_id":4}}
 {"id":4,"title":"mark老师","content":"mark老师NB","tags":["java","面向对 象"],"create_time":1554015482530}
```



## 批量更新

+ 如果原文档不存在，则是创建 
+ 如果原文档存在，则是替换(全量修改原文档)

```
POST _bulk
{"index":{"_index":"article","_type":"_doc","_id":3}}
{"id":3,"title":"图灵徐庶老师","content":"图灵学院徐庶老师666","tags":["jav a", "面向对象"],"create_time":1554015482530}
 {"index":{"_index":"article","_type":"_doc","_id":4}}
{"id":4,"title":"图灵诸葛老师","content":"图灵学院诸葛老师NB","tags": ["java", "面向对象"],"create_time":1554015482530}
```



## 批量删除

```
POST_bulk
{"delete":{"_index":"article","_type":"_doc","_id":3}} 
{"delete":{"_index":"article","_type":"_doc","_id":4}}
```



## 批量部分更新

```
POST _bulk
{"update":{"_index":"article","_type":"_doc","_id":3}} 
{"doc":{"title":"ES大法必修内功"}}

{"update":{"_index":"article","_type":"_doc","_id":4}} 
{"doc":{"create_time":1554018421008}}
```



## 批量读取

批量读取可以分为两种，一种是从多个索引中获取，另一种是从一个索引中获取

_mget

```
GET _mget
{
	"docs":[{
		"_index": "huochai",
		"_id": 3
	},{
		"_index": "article",
		"_id": 3
	}
	]
}


GET /huochai/_mget
{
	"ids": [1,2,3]
}
```

_msearch

与 _bulk 一样，也要先指定索引，再指定查询条件

如果只是查询一个index，我们可以 在url中带上index，这样，如果查该index可以直接用空对象表示。

```
GET /article/_msearch
{}
{"query":{"match_all":{}},"from":0,"size":2} 
{"index":"article"}
{"query":{"match_all":{}}}


GET /_msearch
{"index": "huochai"}
{"query":{"match_all":{}},"from":0,"size":2} 
{"index": "article"}
{"query":{"match_all":{}}}
```

返回的结果其实就是将单个查询的结果组合到一个集合中



# 删除数据

根据条件删除索引，注：此时不是彻底删除，只是打上删除标志，执行时需要注意磁盘空间

```
POST /appraisal_kpi_assessee_v5/_delete_by_query
{
  "query":{
    "match":{
      "planId":"2fb4b3fff87a42f28132c76c53c99db5"
    }
  }
}
```

