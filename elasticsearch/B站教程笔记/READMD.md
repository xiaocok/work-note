# ElasticSearch

## 基础功能

### 索引

类似于数据库的表

#### 创建索引

> PUT /index_name

kibana

```json
PUT /index_name
// 不指定body, 则settings是用的默认值，没有mappings，则没有字段
{
  "aliases" : {
    // 索引别名
  },
  "settings": {
    // 索引设置
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      // 字段映射：字段名称，字段类型；类似于数据库的字段
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "address": {
        // 需要分词
        "type": "text",
        // 使用ik_smart插件进行分词
        "analyzer": "ik_smart"
      }
      "enrolled_date": {
        "type": "date"
      }
    }
  }
}
```

- 索引名称（index_name）

  必须小写字母，可以包含数字和下划线。

- 索引设置（settings）

  分片数量（numberofshards）：一个索引的分片数决定了索引的秉性度和数据分布。

  副本数量（numberifreplicas）：副本提高了数据的可用性和容错能力。

- 映射（mappings）

  字段属性（properties）：定义了索引中的文档的字段及其类型，常用的字段类型包括：text, keyword, integer, date等。

  text：分词，后期模糊搜索

  keyword：不分词，后期精确搜索

  integer：整数

  date：日期

  

postman

```shell
PUT http://ES_IP:9200/index
地址：比上面的kibana多了一个ES的地址

body于上面的json一样
```

 

#### 删除索引

> DELETE /index_name

删除索引会删除该索引下的所有文档和相关的元数据



#### 修改索引

> PUT /index_name/_settings
>
> PUT /index_name/_mapping

 ```json
 PUT /index_name/_settings
 {
   "index": {
     "number_of_replicas": 2
   }
 }
 
 PUT /index_name/_mapping
 {
   // 不存在字段，则增加的字段；已存在字段，则更新字段
   "properties": {
     "grade": {
       "type": "integer"
     }
   }
 }
 ```



#### 查询索引

> GET /index_name

```json
GET index
{
  // 索引名称
  "index" : {
    // 索引别名
    "aliases" : { },
    // 索引映射(字段)
    "mappings" : {
      "properties" : {
        // 字段名称
        "content" : {
          // 字段类型
          // text需要分词，查询时模糊匹配；keyword不分词，查询时精确匹配。
          // content字段是一个text类型的字段，同时它有一个附加的keyword子字段。这个keyword子字段就是用来确保字段值在搜索时不进行分词的。
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        // 字段名称
        "email" : {
          // keyword类型是一种数据类型，通常与string类型一起使用，专门用于需要确保字段值保持完整（即不进行分词）的场景
          // keyword类型时精确匹配
          "type" : "keyword"
        }
      }
    },
    // 索引设置
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        // 分片数
        "number_of_shards" : "1",
        "provided_name" : "index",
        "creation_date" : "1732720372287",
        // 副本数
        "number_of_replicas" : "1",
        "uuid" : "46PPU2q4RiaBfn_pCRZgUg",
        "version" : {
          "created" : "7170599"
        }
      }
    }
  }
}
```

默认content查询时模糊匹配。由于有keyword子字段子字段，如果你想要对content进行精确匹配，你可以使用content.keyword。

例如，如果你想搜索值为New York的字段，你可以这样查询：

```json
{
  "query": {
    // 精确匹配使用term
    "term": {
      "content.keyword": "New York"
    }
  }
}
```





文本类型（text）关键字类型（keyword）区别

> 一切文本类型的字符串可以定义成 “text”文本类型或“keyword”关键字类型两种类型

- `text类型`（文本类型）会使用默认分词器分词，也就是存入的数据会先进行分词，然后将分完词的词组存入索引，当然你也可以为他指定特定的分词器。
  `text类型检索`不是直接给出是否匹配，而是检索出相似度，并按照相似度由高到低返回结果。这样会导致本来我们认为应该查询出来的数据有可能会查询不到。

- `keyword类型`（关键字类型），那么默认就不会对其进行分词，原样存储。当一个字段需要按照精确值进行**过滤、排序、聚合**等操作时, 就应该使用keyword类型.
  `keyword类型检索`，直接被存储为了二进制，检索时我们直接匹配，不匹配就返回false。所以精确匹配可以用keyword。

```json
{
  "mappings": {
    "example_test_type": {
      "dynamic": "false",
      "_all": {
        "enabled": false
      },
      "properties": {
        "userName": {//用户名字：测试人员（可以模糊匹配）
          "type": "text"
        },
        "userPlace": {//用户籍贯：吉林（需要精确匹配）
          "type": "keyword"
        },
        "createTime": {
          "type": "long"
        }
      }
    }
  }
}
```





#### 索引别名

索引别名可以指向一个或多个索引，并且可以在任何需要索引名称的API中使用。

- 在正在运行的集群上的一个索引和另一个索引之间进行透明切换
- 对多个索引进行分组组合
- 在索引中的文档子集上创建”视图“，结合业务场景，缩小了检索范围，提升检索效率



场景：

- 大量数据按日期切分成n个不同的索引(表)，每次查询都需要指定多个索引。（定义一个索引别名，关联这些分割的索引）
- 线上提供服务的索引设计不合理，比如某个字段定义 不合适，怎么保证对外不停服的情况下，不改业务代码的前提下更换索引(更换表)



性能：

索引别名只是物理索引的软连接的名称。如果索引和别名指向相同，则索引和别名的效率是一样的。

查询的时候可以使用索引别名，但是写入和更新的时候，建议使用物理索引。



创建索引的时候指定别名

```json
PUT /index_name
{
  "aliases" : {
    // 索引别名
    "alias_name": {
      //属性
    }
  }
}
```

通过别名可以访问索引

```json
GET alias_name
```



已有索引添加别名

```json
POST /_aliases
{
  "actions" : [
    {
      "add": {
        "index": "index_name",
        "alias": "alias_name"   
      }
    }
  ]
}
```



##### 多索引检索

###### 不使用别名

```json
// 使用都好隔开
PUT logs_202401,logs_202402,logs_202403/_search

// 使用通配符
PUT logs_*/_search
PUT logs_2024*/_search
```



###### 使用别名

```json
// 1. 使用别名关联已有索引
POST /_aliases
{
  "actions" : [
    {
      "add": {
        "index": "logs_202401",
        "alias": "logs_2024"   
      }
    },
    {
      "add": {
        "index": "logs_202402",
        "alias": "logs_2024"   
      }
    },
    {
      "add": {
        "index": "logs_202403",
        "alias": "logs_2024"   
      }
    }
  ]
}

// 2. 使用索引别名进行检索
GET logs_2024/_search
```





### 文档

类似于数据库中的数据/记录



#### 新增文档

##### 新增单个文档

```json
// 不指定ID时，可以使用POST新增。ES会自动生成一个唯一的ID
POST /<index_name>/_doc
{
  "field1": "value1",
  "field2": "value2",
  // 其他字段
}

// 指定唯一ID时，可以使用PUT新增或者更新。如果ID不存在，则新增；如果ID存在，则替换现有文档。
PUT /<index_name>/_doc/<document_id>
{
  "field1": "value1",
  "field2": "value2",
  // 其他字段
}
```

POST和PUT区别：

- 指定文档ID

  PUT必须指定文档ID。如果ID不存在则创建，如果存在，则替换更新。

  POST可以指定ID，也可以不指定ID。不指定ID，则创建，ES会自动生成一个唯一ID。

- 幂等性

  PUT时幂等性的

  POST不是幂等性，多次调用，会创建多个文档

- 更新行为

  PUT会替换整个文档内容

  POST请求更新时可以使用_update的API（详见文档更新部分），这样可以只更新文档的特定字段，而不是替换整个文档



##### 批量新增文档

特性

- 支持一次API调用中，对不同的索引进行操作
- 操作中单条操作失败，并不影响其他操作
- 返回结果包含了每一条的操作结果

操作

```json
POST /<index_name>/_bulk
// 创建
{"create": {"_index": "<index_name>", "_id": "<optional_document_id>"}}
{"field1": "value1", "field2": "value2", ...}
// 创建
{"index": {"_index": "<index_name>", "_id": "<optional_document_id>"}}
{"field1": "value1", "field2": "value2", ...}
// 更新
{"update": {"_index": "<index_name>", "_id": "<document_id>"}}
{"doc": {"field1": "new_value1", "field2": "new_value2", ...}, "_op_type": "update"}
// 删除
{"delete": {"_index": "<index_name>", "_id": "<document_id>"}}
```

- 每一个操作都是一个独立的json对象，这些对象交替出现，形成一个请求体
- 每个index后面跟着的时要创建/替换的索引内容
- update包含了更新文档的内容和操作类型
- delete指明要删除的文档id
- 每个操作对象的开头都必须是create/index/update/delete，并且每个操作之间用一个空行分隔
- 操作不同的索引则POST不指定索引（POST /_bulk），在操作内容中指定具体哪一个索引



_bulk支持四种操作类型

- create：如果文档不存在，则创建；如果文档已存在，则返回错误

- index：用于创建新文档或者替换已有文档
- update：用于更新现有文档
- delete：用于删除指定文档



#### 查询文档

##### 根据id查询文档

查询单个文档

```json
GET /<index_name>/_doc/<document_id>
```



查询多个文档

```json
GET /<index_name>/_mget
{
  // id是字符串存储的。如果id是数字的字符串，这里可以使用数字查询也可以
  "ids": ["id1", "id2", "id3", ...]
}
```



##### 根据关键字查询文档

> 查询文档通常用Query DSL（Domain Specific Language），这是一种基于Json的语言，用于构建复杂的搜索查询

###### 匹配所有文档

```json
GET /<index_name>/_search
{
  "query": {
    "match_all": {}
  }
}
```

###### 文本字段匹配/全文检索

```json
GET /<index_name>/_search
{
  "query": {
    // 查询条件
    "match": {
      "<field_name>": "<query_string>"
    }
  }
}

// 例如
GET /<index_name>/_search
{
  "query": {
    // 查询条件
    "match": {
      "name": "John"
    }
  }
}
```

###### 精确匹配（不分词）

```json
GET /<index_name>/_search
{
  "query": {
    "term": {
      "<field_name>": {
        "value": "<exact_value>"
      }
    }
  }
}
```

###### 范围查询

```json
GET /<index_name>/_search
{
  "query": {
    "range": {
      "<field_name>": {
        "gte": "<lower_bound>",
        "lte": "<upder_bound>"
      }
    }
  }
}
```



#### 更新文档

##### 更新单个文档

```json
POST /<index_name>/_update/<document_id>
{
  "doc": {
    "<field>": "<value>"
  }
}
```



##### 批量更新文档

###### 使用_bulk API

```json
POST /_bulk
{"update": {"_index": "<index_name>", "_id": "<document_id>"}}
{"doc": {"field1": "new_value1", "field2": "new_value2", ...}, "upsert": {"field1": "new_value1", "field2": "new_value2", ...}}
...
```

- doc 文档更新的内容
- upsert 当文档不存在时，插入文档的内容

###### 使用_update_by_query API

根据查询条件更新多个文档。这个操作时原子性的，要么全部更新成功，要么全部都不更新

```json
POST /<index_name>/_update_by_query
{
  "query": {
    // 更新文档的条件
  },
  "script": {
    "source": "ctx._source.field = 'new_value'",
    "lang": "painless"
  }
}

// 例如
POST /employee/_update_by_query
{
  "query": {
    "term": {
      "name": "张三"
    }
  },
  "script": {
    "source": "ctx._source.age = 30"
  }
}
```

- script 定义了如何更新这些文档的字段

###### 并发场景下更新保证线程安全

ES 7.x之后的版本中， _seq_no(序列号)和 _primary_term取代了旧版本的 _version字段，用于控制文档的版本。

- _seq_no代表文档在特定分区中的序列号
- _primary_term代表文档所在主分区的任期编号

这2个字段共同构成了文档的唯一标识符，用于实现乐观锁机制。确保在高并发环境下文档的一致性和正确更新。

在高并发环境下使用乐观锁机制更新文档时，要带上当前文档的 _seq_no和 _primary_term进行更新

```json
POST /employee/_doc/1?if_seq_no=12&if_primary_term=1
{
  "name": "张三",
  "sex": 1,
  "age": 25
}
```

如果 _seq_no和 _primary_term不对，会抛出版本冲突异常



#### 删除文档

##### 删除单个文档

```json
DELETE /<index_name>/_doc/<document_id>
```



##### 批量删除文档

###### 使用_bulk API

```json
// 不同索引
POST /_bulk
{"delete": {"_index": "<index_name>", "_id": "<document_id>"}}
{"delete": {"_index": "<index_name>", "_id": "<document_id>"}}
{"delete": {"_index": "<index_name>", "_id": "<document_id>"}}

// 相同索引
POST /<index_name>/_bulk
{"delete": {"_id": "<document_id>"}}
{"delete": {"_id": "<document_id>"}}
{"delete": {"_id": "<document_id>"}}
```

###### 使用_delete_by_query API

根据查询条件删除文档

```json
POST /<index_name>/_delete_by_query
{
  "query": {
    "<your_query>"
  }
}

// 例如
POST /employee/_delete_by_query
{
  "query": {
    "match": {
      "address": "广州"  
    }
  }
}
```





### 参考

[【2024版ES8.x教程】这可能是B站唯一能将ElasticSearch8.x讲明白的教程](https://www.bilibili.com/video/BV1TjpcehEti)

[ElasticSearch中字符串keyword和text类型区别](https://baijiahao.baidu.com/s?id=1763386735571186685)