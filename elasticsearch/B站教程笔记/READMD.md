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
      },
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



##### 查询中：GET和POST

GET 请求
用途：主要用于从 Elasticsearch 中检索数据。

```json
// 获取索引信息：获取索引的设置、映射等信息。
GET /my_blogs

// 获取文档：通过文档 ID 获取单个文档。
GET /my_blogs/_doc/blog1
```



POST 请求
用途：主要用于向 Elasticsearch 发送复杂的查询请求、索引文档、更新文档等操作。

```json
// 搜索查询：使用复杂的查询 DSL 进行搜索。
POST /my_blogs/_search
{
    "query": {
        "match": {
            "title": "Learning Hadoop"
        }
    }
}

// 索引文档：虽然可以使用 POST 来索引文档，但更常见的是使用 PUT。
POST /my_blogs/_doc/
{
    "title": "Learning Go",
    "content": "learning Go",
    "blog_comments_relation": {
        "name": "blog"
    }
}
```



主要区别
安全性：
		GET 请求通常用于只读操作，不会修改数据。
		POST 请求可以用于读写操作，可能会修改数据。
请求体：
		GET 请求的参数通常通过 URL 传递，不适合传递复杂的数据结构。
		POST 请求可以在请求体中传递复杂的数据结构，适合复杂的查询和操作。
幂等性：
		GET 请求是幂等的，多次请求相同的 URL 应该返回相同的结果。
		POST 请求不是幂等的，多次请求相同的 URL 可能会导致不同的结果（例如，插入相同的数据可能会导致重复记录）。
URL 长度限制：
		GET 请求的 URL 长度有限制，不适合传递大量数据。
		POST 请求没有 URL 长度限制，适合传递大量数据。

**兼容：**

​        ES做了一个兼容，及时使用GET请求，传递了POST的Body数据，ES也会从body中读取数据，兼容了这一场景。GET和POST的效果除了http层面的一些限制(如url长度等)，其他的看起来就一样了，但是GET方式传递POST的body方式不符合Rest风格，推荐还是按Rest风格来使用GET和POST。

```json
// 使用 GET 进行简单查询：
GET /my_blogs/_search?q=title:Learning

// 使用 POST 进行复杂查询：
POST /my_blogs/_search
{
    "query": {
        "bool": {
            "must": [
                { "match": { "title": "Learning" } },
                { "match": { "content": "Hadoop" } }
            ]
        }
    }
}
```

总结来说，GET 请求适用于简单的数据检索，而 POST 请求适用于复杂的查询和操作。在 Elasticsearch 中，大多数复杂的查询和操作都会使用 POST 请求。



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



## 最佳实践

### 关联关系

#### 嵌套对象（Nested Object）

- Nested类型适用于**一对少量**、子文档偶尔更新，查询频繁的场景。
- 如果要索引**对象数组**并**保持数组中每个对象的独立性**，则使用Nested类型而不是Object数据类型。
- Nested类型的优点是可以将父子文档的两部分数据关联起来（例如博客与论坛），可以基于Nested类型做任何查询。缺点是查询相对较慢，更新子文档时需要更新整篇文档。



##### 对象类型（Object）

- 在每一个博客的文档中都保留作者的信息
- 如果作者信息发生变化，需要修改相关的博客文档

```json
// 创建索引
PUT /blog
{
    "mappings": {
        // properties是对象类型，表示是一个对象
        "properties": {
            "context": {
                "type": "text"
            },
            "time": {
                "type": "date"
            },
            "user": {
                // properties是对象类型，表示是一个对象
                "properties": {
                    "city": {
                        "type": "text"
                    },
                    "userid": {
                        "type": "long"
                    },
                    "username": {
                        "type": "keyword"
                    }
                }
            }
        }
    }
}

// 创建文档
PUT /blog/_doc/1
{
    "content": "I like ElasticSearch",
    "time": "2022-01-01T00:00:00",
    // 嵌套对象，以json存对象
    "user": {
        "userid": 1,
        "username": "Fox",
        "city": "ChangSha"
    }
}

// 查询
POST /blog/search
{
    "query": {
        // 组合查询bool+must，类似于数据库的And，多个条件同时满足
        "bool": {
            "must": [
                {"match": {"context": "ElasticSearch"}},
                {"match": {"user.username": "Fox"}},
            ]
        }
    }
}
```

##### 嵌套类型（Nested）

- Nested数据类型：允许对象数组中的对象被独立索引
- 使用Nested和properties关键字，将所有actors索引到多个分隔的文档
- 在内部Nested文档会被保存在两个Lucene文档中，在查询时做Join处理

错误的场景：

包含对象数组的文档

```json
PUT /my_movies
{
    "mappings": {
        "properties": {
            "actors": {
                // properties是对象类型，表示是一个对象
                "properties": {
                    "first_name": {
                        "type": "keyword"
                    },
                    "last_name": {
                        "type": "keyword"
                    }
                }
            },
            "title": {
                "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword",
                        "ignore_above": 256
                    }
                }
            }
        }
    }
}

PUT /my_movies/_doc/1
{
    "title": "Speed",
    "actors": [
        {
            "first_name": "Keanu",
            "last_name": "Reeves"
        },
        {
            "first_name": "Dennis",
            "last_name": "Hopper"
        }
    ]
}

// 查询
POST /my_movies/_search
{
    "query": {
        "bool": {
            "must": [
                {"match": {"actors.first_name": "Keanu"}},
                {"match": {"actors.last_name": "Hopper"}},
            ]
        }
    }
}
// 返回结果是两条数据
// 查询是跟mysql的json查询类似，效果等同于：actors.first_name IN ["Keanu", "Dennis"] AND actors.last_name IN ["Reeves", "Hopper"]
```

调整为嵌套类型（Nested）

```json
PUT /my_movies
{
    "mappings": {
        "properties": {
            "actors": {
                // nested表示是嵌套类型
                "type": "nested",
                "properties": {
                    "first_name": {"type": "keyword"},
                    "last_name": {"type": "keyword"}
                }
            },
            "title": {
                "type": "text",
                "fields": {"keyword": {"type": "keyword","ignore_above": 256}}
            }
        }
    }
}

// 查询
POST /my_movies/_search
{
    "query": {
        "bool": {
            "must": [
                {"match": {"title": "Speed"}},
                {
                    "nested": {
                        "path": "actors",
                        "query": {
                            "bool": {
                            	"must": [
                                	{"match": {"actors.first_name": "Keanu"}},
		                			{"match": {"actors.last_name": "Reeves"}}
                                ]
                            }
                        }
                    }
                }
            ]
        }
    }
}
```



#### Join父子文档类型

- Join类型用于在同一索引的文档中创建父子关系。
- Join类型适用于子文档数据量明显多于父文档的数据量的场景。该场景存在**一对多量**的关系，子文档更新频繁。例如：一个产品和供应商之间就是一对多的关联关系。
- 当使用父子文档时，使用has_child或者has_parent做父子关系查询。
- Join类型的优点是父子文档可以独立更新。缺点是维护Join关系需要占用部分内存，查询较Nested类型更耗资源。



- 对象和Nested对象的局限性：每次更新，可能需要重新索引整个对象（包括根对象和嵌套对象）
- ES提供了类似关系型数据库中Join的实现。使用Join数据类型实现，可以通过维护Parent/Child的关系，从而分离两个对象
  - 父文档和子文档是两个独立的文档
  - 更新父文档无需重新索引子文档。子文档被添加，更新或者删除也不会影响到父文档个其他子文档

```json
// 设置Parent/Child Mappings
PUT /my_blogs
{
    "settings": {
        "number_of_shards": 2
    },
    "mappings": {
        "properties": {
            // 博客和评论的关系
            "blog_comments_relation": {
                // 指明join类型
                "type": "join",
                "relations": {
                    // blog是Parent名称
                    // comment是Child名称
                    "blog": "comment"
                }
            },
            // 博客内容
            "comment": {
                "type": "text"
            },
            // 博客标题
            "title": {
                "type": "keyword"
            }
        }
    }
}
```

索引父文档 / 插入父文档数据

```json
// blog1 父文档id
PUT /my_blogs/_doc/blog1
{
    "title": "Learning ElasticSearch",
    "content": "learning ELK",
    // 申明文档的类型
    "blog_comments_relation": {
        // 指明父文档的字段
        "name": "blog"
    }
}

PUT /my_blogs/_doc/blog2
{
    "title": "Learning Hadoop",
    "content": "learning Hadoop",
    "blog_comments_relation": {
        "name": "blog"
    }
}
```

索引子文档 / 插入子文档数据

```json
// comment1 子文档id
// 指定routing，确保和父文档索引到相同的分片
PUT /my_blogs/_doc/comment1?routing=blog1
{
    // 子文档类型在mappings中未定义。
    "content": "I am learning ELK",
    "username": "Jack",
    "blog_comments_relation": {
        // 指明子文档的字段
        "name": "comment",
        // 父文档id。额外需要指定父文档id
        "parent": "blog1"
    }
}

PUT /my_blogs/_doc/comment2?routing=blog2
{
    "content": "I like Hadoop!!!!!",
    "username": "Jack",
    "blog_comments_relation": {
        "name": "comment",
        "parent": "blog2"
    }
}

PUT /my_blogs/_doc/comment3?routing=blog2
{
    "content": "Hello Hadoop",
    "username": "Bob",
    "blog_comments_relation": {
        "name": "comment",
        "parent": "blog2"
    }
}
```

- 父文档和子文档必须存在相同的分片上，能够确保查询Join的性能
- 当指定子文档时候，必须指定它的父文档id。使用routing参数来保证，分配到相同的分片

子文档未在mappings中定义说明

- 动态映射：ElasticSearch 默认启用了动态映射功能。这意味着当你索引一个文档时，如果其中包含未在映射中预先定义的新字段，ElasticSearch 会自动检测这些字段的数据类型并将其添加到映射中。因此，尽管你在创建索引时没有显式定义 username 字段，但当你第一次插入包含 username 字段的文档时，ElasticSearch 自动为该字段创建了映射。
- 字段类型推断：当动态添加新字段时，ElasticSearch 会根据字段值的类型进行推断。例如，"Jack" 和 "Bob" 被识别为字符串，所以 username 字段会被设置为 text 类型（默认情况下）。同时还会创建一个 .keyword 子字段用于精确匹配查询。
- 子文档结构：虽然你在映射中只定义了 blog_comments_relation 来表示父子关系，但这并不限制你可以向子文档（如评论）添加其他自定义字段。只要这些字段符合 JSON 格式要求，就可以被正常索引和查询。

完整定义参考

```json
PUT /my_blogs
{
    "settings": {
        "number_of_shards": 2
    },
    "mappings": {
        "properties": {
            "blog_comments_relation": {
                "type": "join",
                "relations": {
                    "blog": "comment"
                }
            },
            // 父文档字段
            "comment": {
                "type": "text"
            },
            // 父文档字段
            "title": {
                "type": "keyword"
            },
            // 子文档字段
            "content": {
                "type": "text"
            },
            // 子文档字段
            "username": {
                "type": "keyword"
            }
        }
    }
}
```

查询

```json
// 查询所有文档：父文档+子文档
POST /my_blogs/_search

// 根据父文档id查询
GET /my_blogs/_doc/blog2

// Parent id查询
POST /my_blogs/_search
{
    "query": {
        "parent_id": {
            "type": "comment",
            "id": "blog2"
        }
    }
}

// Has Chile查询，返回父文档
POST /my_blogs/_search
{
    "query": {
        "has_child": {
            "type": "comment",
            "query": {
                "match": {
                    "username": "Jack"
                }
            }
        }
    }
}

// Has Parent查询，返回相关的子文档
POST /my_blogs/_search
{
    "query": {
        "has_parent": {
            "parent_type": "blog",
            "query": {
                "match": {
                    "title": "Learning Hadoop"
                }
            }
        }
    }
}
```



#### 宽表冗余存储

- 宽表适用于**一对多**或者**多对多**的关联关系。
- 宽表的优点是速度快。缺点是索引更新或者删除数据时，应用程序不得不处理宽表的冗余数据；并且由于冗余存储，某些搜索和聚合操作的结果可能不准确。



#### 业务端关联

- 普遍使用的技术，即在应用接口层面处理关联关系。一般建议在存储层面使用两个独立的索引存储，在实际业务层面将分为两次请求完成。
- 业务端关联适用于**数据量少的多表关联业务场景**。数据量少时，用户体验好；而数据量多时，两次查询耗时比较长，反而影响用户体验。



#### 多表关联方案对比

在ElasticSearch开发实战中，对于多表关联的设计需要突破关系型数据库的设计思维定式。不建议在ElasticSearch中做多表关联操作，尽量在设计时使用扁平表文档模型，或者尽量将业务转化为没有关联关系的文档形式。

|          | Nested嵌套类型                                       | Join父子文档类型                               | 宽表冗余存储               | 业务端关联                                 |
| -------- | ---------------------------------------------------- | ---------------------------------------------- | -------------------------- | ------------------------------------------ |
| 优点     | 文档储存在一起，读取性能高                           | 父子文档可以独立更新，互不影响                 | 以空间换时间               | 数据量少时，用户体验好                     |
| 缺点     | 更新嵌套的子文档时，需要更新整个父文档，查询相对较慢 | Join关系的维护也耗费内存。读取性能比Nested还差 | 字段冗余造成存储空间的浪费 | 数据量多，两次查询耗时比较长，影响用户体验 |
| 适用场景 | 对少量、子文档偶尔更新、查询频繁                     | 子文档更新频繁                                 | 一对多或者多对多           | 数据量少                                   |



### 避免过多的字段

- 一个文档中，最好避免大量的字段

  - 字段过多，数据不易维护
  - Mappings信息保存在Cluster State中，数据量过大，对集群新能会有影响
  - 删除或者修改数据需要reindex

- 默认最大字段数是1000，可以设置index.mapping.total_fields.limit限制最大字段数

  - 生产环境中，尽量不要打开Dynamic（默认为True），可以使Strict控制新增字段的加入（插入数据时，如果开启Dynamic，新增的不存在的字段，会增加字段数）

    - true：未知字段会被自动加入，默认值
    - false：新字段不会被索引，但是会保存在_source
    - strict：新增字段不会被索引，文档写入失败

    ```json
    PUT /users
    {
        "mappings": {
            "dynamic": "strict",
            "properties": {
                "name": {
                    "type": "text"
                },
                "address": {
                    "type": "object",
                    "dynamic": "true"
                }
            }
        }
    }
    ```

    

### 避免正则，通配符，前缀查询

正则、通配符、前缀查询属于Term查询，但是性能不够好。特别是将通配符放在开头，会导致新能的灾难



### 避免空值引起的聚合不准

```json
// Not Null解决聚合的问题
PUT /scores
{
    "mappings": {
        "propreties": {
            "score": {
                "type": "float",
                "null_value": 0
            }
        }
    }
}

PUT /scores/_doc/1
{
    "score": 100
}

PUT /scores/_doc/2
{
    "score": null
}

POST /scores/_search
{
    "size": 0,
    "aggs": {
        "avg": {
            "avg": {
                "field": "score"
            }
        }
    }
}
```



### 为索引的Mapping加入Meta信息

- Mappings设置非常重要，需要从两个维度进行考虑
  - 功能：索引，聚合，排序
  - 性能：存储的开销；内存的开销；搜索的新能
- Mappings设置是一个迭代的过程
  - 加入新的字段很容易（必要时需要update_by_query）
  - 更新删除字段不允许（需要Reindex重建数据）
  - 最好能对Mappings加入Meta信息，更好的进行版本管理
  - 可以考虑将Mapping文件上传git进行管理

```json
PUT /my_index
{
    "mappings": {
        "_meta": {
            "index_version_mapping": "1.1"
        }
    }
}
```



## 分词器

### analysis-icu

这个分析器基于 ICU（International Components for Unicode）库，支持多语言文本的分词和处理。

安装插件

```shell
bin/elasticsearch-plugin install analysis-icu
```

**使用 `icu_analyzer`**

`icu_analyzer` 是 `analysis-icu` 插件提供的默认分析器。你可以在索引映射或分析 API 中直接使用它。

示例：在分析 API 中使用 `icu_analyzer`

```json
POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "这是一个测试文本"
}
```

示例：在索引映射中使用 `icu_analyzer`

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "icu_analyzer"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text"
      }
    }
  }
}
```

**ICU 分词器（`icu_tokenizer`）**

如果你需要更细粒度的控制，可以使用 `icu_tokenizer` 分词器，并结合其他过滤器（如 `icu_normalizer`、`icu_folding` 等）来创建自定义分析器。

**示例：自定义分析器**

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "tokenizer": "icu_tokenizer",
          "filter": [
            "icu_normalizer",
            "icu_folding"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

**ICU 过滤器**

`analysis-icu` 插件还提供了多种过滤器，可以用于自定义分析器：

- **`icu_normalizer`**: 对文本进行 Unicode 规范化。

- **`icu_folding`**: 将字符折叠为 ASCII 等效字符（如去除变音符号）。
- **`icu_collation`**: 支持基于 ICU 的排序规则。

示例：使用 `icu_folding` 过滤器

```json
POST _analyze
{
  "tokenizer": "icu_tokenizer",
  "filter": ["icu_folding"],
  "text": "Café au lait"
}
```

**总结**

- **默认分析器**：使用 `icu_analyzer`。
- **自定义分析器**：结合 `icu_tokenizer` 和 ICU 过滤器（如 `icu_normalizer`、`icu_folding`）创建自定义分析器。
- **分析 API**：可以通过 `_analyze` API 测试分词效果。



### analysis-smartcn

是 Elasticsearch 的一个官方插件，专门用于处理中文文本的分词和分析。它基于 **Smart Chinese Analysis** 算法，适合对简体中文文本进行分词。

**插件安装**

```shell
bin/elasticsearch-plugin install analysis-smartcn
```

**docker**

```shell
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:8.10.0 \
  sh -c "bin/elasticsearch-plugin install analysis-smartcn && /usr/local/bin/docker-entrypoint.sh"
```

**使用 `smartcn` 分析器**

`analysis-smartcn` 插件提供了一个默认的分析器 `smartcn`，可以直接用于中文文本的分词。

示例：在分析 API 中使用 `smartcn` 分析器

```json
POST _analyze
{
  "analyzer": "smartcn",
  "text": "这是一个测试文本"
}
```

**在索引映射中使用 `smartcn` 分析器**

你可以将 `smartcn` 分析器设置为字段的默认分析器。

示例：创建索引并使用 `smartcn` 分析器

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "smartcn"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text"
      }
    }
  }
}
```

插入文档并测试分词

```json]
POST my_index/_doc/1
{
  "content": "这是一个测试文本"
}

POST my_index/_analyze
{
  "field": "content",
  "text": "这是一个测试文本"
}
```

**自定义 `smartcn` 分析器**

如果需要更细粒度的控制，可以结合 `smartcn_tokenizer` 和其他过滤器（如 `stop`、`lowercase` 等）创建自定义分析器。

示例：自定义分析器

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_smartcn_analyzer": {
          "tokenizer": "smartcn_tokenizer",
          "filter": [
            "lowercase",
            "stop"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_custom_smartcn_analyzer"
      }
    }
  }
}
```

测试自定义分析器

```json
POST my_index/_analyze
{
  "analyzer": "my_custom_smartcn_analyzer",
  "text": "这是一个测试文本"
}
```



### analysis-ik

是一个流行的 Elasticsearch 中文分词插件，专门为中文文本设计。它提供了比 `analysis-smartcn` 更强大的分词功能，支持细粒度和粗粒度分词模式，适合处理复杂的中文分词需求。



#### **安装 `analysis-ik` 插件**

`analysis-ik` 是第三方插件，需要手动安装。以下是安装步骤：

**方法 1：手动安装**

1. 下载与 Elasticsearch 版本匹配的 `analysis-ik` 插件：

   - 从 [GitHub 发布页面](https://github.com/medcl/elasticsearch-analysis-ik/releases) 下载。
   - 例如，Elasticsearch 8.x 版本可以下载 `elasticsearch-analysis-ik-8.x.x.zip`。

2. 将插件文件解压到 Elasticsearch 的 `plugins` 目录：

   ```bash
   unzip elasticsearch-analysis-ik-8.x.x.zip -d /path/to/elasticsearch/plugins/ik
   ```

3. 重启 Elasticsearch。

**方法2：自动安装**

```shell
bin/elasticsearch-plugin install https://get.infini.cloud/elasticsearch/analysis-ik/8.4.1
bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.10.0/elasticsearch-analysis-ik-8.10.0.zip
```

**方法 3：在 Docker 中安装**

> 优于方法4，不用每次运行都安装一次

如果你使用 Docker 运行 Elasticsearch，可以在启动容器时安装插件：

```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -v /path/to/plugins:/usr/share/elasticsearch/plugins \
  docker.elastic.co/elasticsearch/elasticsearch:8.10.0
```

然后将 `analysis-ik` 插件解压到 `/path/to/plugins/ik` 目录。

**方法4：在 Docker 中安装**

```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -v /path/to/plugins:/usr/share/elasticsearch/plugins \
  docker.elastic.co/elasticsearch/elasticsearch:8.10.0 \
  sh -c "bin/elasticsearch-plugin install https://get.infini.cloud/elasticsearch/analysis-ik/8.4.1 && /usr/local/bin/docker-entrypoint.sh"
```

#### **使用 `ik` 分析器**

`analysis-ik` 插件提供了两种分词模式：

- **`ik_smart`**：细粒度分词，适合精确分词。
- **`ik_max_word`**：粗粒度分词，适合最大词量分词。

示例：在分析 API 中使用 `ik_smart` 和 `ik_max_word`

```json
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国"
}
```

```json
POST _analyze
{
  "analyzer": "ik_max_word",
  "text": "中华人民共和国"
}
```

**在索引映射中使用 `ik` 分析器**

你可以将 `ik_smart` 或 `ik_max_word` 设置为字段的默认分析器。

示例：创建索引并使用 `ik_smart` 分析器

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "ik_smart"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text"
      }
    }
  }
}
```

插入文档并测试分词

```json
POST my_index/_doc/1
{
  "content": "中华人民共和国"
}

POST my_index/_analyze
{
  "field": "content",
  "text": "中华人民共和国"
}
```



```json
PUT my_index
{
    "mappings": {
        "properties": {
            "title": {
                "type": "text",
                // 指定在 “索引文档时” 使用的分词器为 ik_max_word。ik_max_word 是 IK 分词器的一种模式，它会将文本进行最细粒度的分词，尽可能多地切分出词语。
                "analyzer": "ik_max_word",
                // 指定在 “搜索时” 使用的分词器为 ik_smart。ik_smart 是 IK 分词器的另一种模式，它会进行较粗粒度的分词，以提高搜索效率。
                "search_analyzer": "ik_smart"
            },
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            },
            "author": {
                "type": "keyword"
            },
            "tags": {
                "type": "keyword"
            },
            "date": {
                "type": "date",
                "format": "yyyy-MM-dd'T'HH:mm:ss"
            }
        }
    }
}
```



#### **自定义词典**

`analysis-ik` 支持自定义词典，可以根据业务需求添加新词或停用词。

**步骤：**

1. 在 `config/analysis-ik/` 目录下创建自定义词典文件，例如 `custom_word.dic`。

2. 在 `config/analysis-ik/IKAnalyzer.cfg.xml` 中配置自定义词典：

   ```xml
   <properties>
       <comment>IK Analyzer 扩展配置</comment>
       <entry key="ext_dict">custom_word.dic</entry>
       <entry key="ext_stopwords">stop_word.dic</entry>
   </properties>
   ```

   运行 HTML

3. 重启 Elasticsearch。



### **`smartcn`、`ik` 和 `icu` 的对比**

- **`analysis-smartcn`**：
  - 适合通用中文分词。
  - 分词效果较为简单，适合不需要复杂分词的场景。
- **`ik` 分词器**：
  - 支持细粒度和粗粒度分词。
  - 适合需要更精确分词的中文场景。
- **`icu` 分词器**：
  - 支持多语言分词，但对中文的分词效果不如 `smartcn` 和 `ik`。



## ES 复杂查询/高级查询 DSL



### match_all匹配所有文档

匹配索引中的所有文档，而不考虑任何的查询条件。默认只返回10条。

**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "match_all": {}
    }
}
```

**高级语法**

添加额外的参数控制搜索的结果显示

- size：返回文档的数量
- from：开始返回的文档位置
- sort：排序规则
- source：指定返回的字段：默认True。

```json
// 返回前10个文档，按文档的评分排序
POST /scores/_search
{
    "query": {
        "match_all": {}
    },
    "from": 0,
    "size": 10,
    "sort": [
        {"_score": {"order": "desc"}},// 根据_score排序
        {"title": "desc"}// 根据title字段排序
    ]
}

// 不查看源数据，只查看元数据
GET /<index_name>/_search
{
    "query": {
        "match_all": {}
    }，
    // 不返回_source内的字段
    "_source": false
}

// 返回指定字段
{
    "query": {
        "match_all": {}
    },
    // 只返回_source内部指定的字段
    "_source": ["field1", "field2"]
}

// 只返回以obj.开头的字段
// 返回指定字段
{
    "query": {
        "match_all": {}
    },
    "_source": "obj.*"
}
```



### ▲精确匹配检索

精确匹配指的是搜索内容不经过文本分析直接用于文本匹配**（不需要分词：精确匹配/模糊匹配都不分词。全文检索会分词）**，这个过程类似于数据库的SQL查询，搜索的对象大多是索引的非text字段。此类检索主要应用于结构化数据，如ID、状态、和标签等。



#### 精确匹配

##### ▲term-单字段精确匹配查询

主要应用于单子段精确匹配的场景。term检索的是非text类型，但是用于text类型时并不会报错，但是检索结果一般不会达到预期。所以在实战中，尽量避免term用于text类型的检索。

```json
GET /<index_name>/_search
{
    "query": {
        "term": {
            "{field.keyword}": {
                "value": "your_exact_value"
            }
        }
    }
}

GET /employee/_search
{
    "query": {
        "term": {
            // 类型为keyword类型
            "name": {
                "value": "张三"
            }
        }
    }
}

GET /employee/_search
{
    "query": {
        "term": {
            // 类型为text，使用.keyword实现不分词精确匹配
            "address.keyword": {
                "value": "广州白云山公园"
            }
        }
    }
}
```

term处理多值字段(数组)时，term查询时包含，不是等于

```json
POST /people/_bulk
{"index": {"_id": 1}}
{"name": "小明", "interest": ["跑步", "篮球"]}
{"index": {"_id": 2}}
{"name": "小红", "c": ["跳舞", "画画"]}
{"index": {"_id": 3}}
{"name": "小丽", "interest": ["跑步", "唱歌", "跑步"]}

POST /people/_search
{
    "query": {
        "trem": {
            "interest.keyword": {
                // 返回interest包含了跑步的数据；匹配到数据的数组中的一个元素即可返回
                "value": "跑步"
            }
        }
    }
}
```

term精确匹配去掉算分

```json
GET /employee/_search
{
    "query": {
        // Constant Score（忽略算分，不计算分数，分数返回都是1）将查询转换成一个Filtering，避免算分，并利用缓存，提高性能
        "constant_score": {
            // 将Query转成Filter，忽略TF-IDF计算，避免相关算分的开销
            // Filter可以有效的利用缓存
            "filter": {
                "term": {
                    "address.keyword": "广州白云山公园"
                }
            }
        }
    }
}
```



##### ▲terms-多字段精确匹配查询

主要用于多值精确匹配的场景。允许在单个查询中指定多个词条来进行精确匹配。适合从文档中查找包含多个特定值的字段。例如筛选包含多个特定标签或状态的项目。

terms允许指定一个特定的字段，并匹配该字段的多个精确值。

```json
GET /<index_name>/_search
{
    "query": {
        "terms": {
            "<field_name>": [
                "value1",
                "value2",
                "value3"
            ]
        }
    }
}

POST /people/_search
{
    "query": {
        "trems": {
            "interest.keyword": [
                "画画", "唱歌"
            ]
        }
    }
}
```



##### ▲range-范围查询

针对指定字段值在给定的范围内的文档检索。适合对数字、日期或者其他可排序数据类型。

range检索支持多种比较符操作，例如 大于(gt)、大于等于(gte)、小于(lt)和小于等于(lte)等。



**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "range": {
            "<field_name>": {
                "gte": <lower_bound>,
                "lte": <upper_bound>,
                "gt": <greater_than_bound>,
                "lt": <less_than_bound>
            }
        }
    }
}

POST /employee/_search
{
    "query": {
        "range": {
            "age": {
                "gte": 25,
                "lte": 28
            }
        }
    }
}
```



**日期范围**

```json
PUT /notes{
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "title": {"type": "text"},
            "content": {"type": "text"},
            "created_at": {"type": "date", "format": "yyyy-MMM-dd HH:mm:ss"}
        }
    }
}

PUT /notes/_bulk
{"index": {"_id": 1}}
{"title": "Note 1", "content": "This is the first note.", "create_at": "2023-07-01 12:00:00"}
{"index": {"_id": 2}}
{"title": "Note 2", "content": "This is the second note.", "create_at": "2023-07-05 15:30:00"}
{"index": {"_id": 3}}
{"title": "Note 3", "content": "This is the third note.", "create_at": "2023-07-10 08:45:00"}
{"index": {"_id": 4}}
{"title": "Note 4", "content": "This is the forth note.", "create_at": "2023-07-15 20:15:00"}

// 查询
POST /notes/_search
{
    "query": {
        "range": {
            "created_at": {
                "gte": "2023-07-05 00:00:00",
                "lte": "2023-07-10 23:59:59"
            }
        }
    }
}
```

ElasticSearch支持日期数学表达式，允许在查询和聚合中使用相对的时间点。

- now：当前时间点
- now-1d：从当前时间点向前推1天的时间点
- now-1w：从当前时间点向前推1周时间点
- now-1m：从当前时间点向前推1个月时间点
- now-1y：从当前时间点向前推1年时间点
- now+1h：从当前时间点向后推1个小时的时间点

```json
POST /product/_bulk
{"index": {"_id": 1}}
{"price": 100, "date": "2023-01-01", "productId": "XHDK-1293"}
{"index": {"_id": 2}}
{"price": 200, "date": "2022-01-01", "productId": "KDKE-5421"}

GET /product/_search
{
    "query": {
        "range": {
            "date": {
                "gte": "now-2y"
            }
        }
    }
}
```



##### exists-是否存在查询

在ElasticSearch中筛选具有特定字段值的文档。这种查询类型适用于检查文档中是否存在某个字段。或者该字段是否包含空值。

通过使用exists检索，你可以有效过滤缺少关键字信息的文档，从而专注于包含所需数据的文档。

应用场景包括但不限于数据完整性检查、查询特定属性的文档以及对可选字段进行筛选等。

**基本语法**

```json
GET /<index_name>/search
{
    "query": {
        "exists": {
            "field": "missing_field"
        }
    }
}

// 查询索引库中存在remark字段的文档
GET /employee/_search
{
    "query": {
        "exists": {
            "field": "remark"
        }
    }
}
```



##### ids-根据一组id查询

允许基于给定的ID组快速召回相关数据，从而实现高效的文档检索。

**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "ids": {
            "values": ["id1", "id2", "id3", ......]
        }
    }
}
```



#### 模糊匹配

##### prefix-前缀查询

prefix会对分词后的term进行前缀搜索。

- 不会对要检索的字符串分词，传入的前缀就是要查找的前缀
- 默认状态下，前缀查询不做相关性分数计算，它只将所有匹配的文档返回，然后赋予所有相关分数为1

**原理：**

需要遍历所有倒排索引（性能较差），并比较每一个分词项是否以所搜索的前缀开头。

**场景：**

通常用于自动补全或者搜索功能，其中用户输入的搜索词可能是更长文本的一部分。

**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "prefix": {
            "your_field_name": {
                "value": "your_prefix_string"
            }
        }
    }
}
```

需要注意的是，这种查询方式仅适用于关键字类型（keyword）的字段。

```json
// 测试数据为address=广州白云山公园

// 接口调用不报错，但是查询不到数据
// 因为address被分词了，而输入的prefix不会被分词，所以查询的数据匹配不到查询条件
GET /employee/_search
{
    "query": {
        "prefix": {
            "address": {
                "value": "广州白云山"
            }
        }
    }
}

// 可以返回数据，使用不分词的数据进行查询后筛选
GET /employee/_search
{
    "query": {
        "prefix": {
            "address.keyword": {
                "value": "广州白云山"
            }
        }
    }
}
```



##### wildcard-通配符查询

- 星号(*)：表示零活多个自字符，可用于匹配任意长度的字符串
- 问号(?)：表示一个字符，用于匹配任意单个字符

**场景：**

适用于对部分已知内容的文本字段进行模糊检索。例如：在文件名或产品型号等具有一定规律的字符串中，使用通配符检索可以方便地找到满足特定模式的文档。

注意：通配符匹配可能会导致较高的计算负担，因此在实际应用中应谨慎使用，尤其是涉及到大量文档的情况下。

**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "wildcard": {
            "your_field_name": {
                "value": "your_search_pattern"
            }
        }
    }
}

GET /employee/_search
{
    "query": {
        "prefix": {
            "address.keyword": {
                "value": "*州*公园"
            }
        }
    }
}
```

 

##### regexp-正则匹配查询

是一种基于正则表达式的检索方式。

虽然该检索方式功能强大，但建议在非必要情况下避免使用，以保证性能的高效和稳定。

```json
GET /<index_name>/_search
{
    "query": {
        "regexp": {
            "your_field_name": {
                "value": "your_search_pattern"
            }
        }
    }
}

GET /employee/_search
{
    "query": {
        "regexp": {
            "remark": {
                "value": "java.*"
            }
        }
    }
}
```

> .*表示在java后可以跟随任意数量的任意字符



##### fuzzy-支持编辑距离的模糊查询

是一种强大的检索功能，它能够在用户输入内容存在拼写错误或者上下文不一致时，仍然返回与搜索词相似的文档。通过使用编辑距离算法来度量输入词与文档中词条的相似程度，模糊查询在保证搜索结果相关的同时，有效地提高了搜索容错能力。

> 编辑距离是指一个单词转为为另外一个单词需要编辑单字符的次数。
>
> 如中文集团到中威集团编辑距离是1，只需要修改一个字符。
>
> 如果fuzziness值在这里设置成为2，会把编辑距离为2的东东集团也查询出来。

```json
GET /<index_name>/_search
{
    "query": {
        "fuzzy": {
            "your_field": {
                "value": "search_term",
                "fuzziness": "AUTO",
                "prefix_length": 1
            }
        }
    }
}

GET /employee/_search
{
    "query": {
        "fuzzy": {
            "address": {
                // 故意写错了一个云->运
                "value": "白运山",
                "fuzziness": 1
            }
        }
    }
}
```

- fuzziness：用于编辑距离的设置，其默认值为AUTO，支持数值为0，1，2。如果设置越界会报错。
- prefix_length：搜索词的前缀长度，在此长度内不会应用模糊匹配。默认是0，即整个词都会模糊匹配。1表示"白"需要精确匹配，"运山"模糊搜索。2表示"白运"需要精确匹配，"山"模糊搜索。3表示"白运山"需要精确匹配。



##### term set-用于解决多值字段中的文档匹配问题

是ES中强大的检索类型，主要用于解决多值字段中的文档匹配问题，在处理具有多个属性、分类或标签的复杂数据时非常有用。

场景：在处理多值字段和特定匹配条件时具有很大的优势。它适用于标签系统、搜索引擎、电子商务系统、文档管理系统和技能匹配等场景。

**基本语法**

可以检索至少匹配一定数量给定词项的文档，其中匹配的数量可以是固定值，也可以是基于另一个字段的动态值。基本语法如下：

```json
GET /<index_name>/_search
{
    "query": {
        "terms_set": {
            "<field_name>": {
                "terms": ["<term1>", "<term2>", ...],
                "minimum_should_match_field": "minimum_should_match_field_name"
                OR 
                "minimum_should_match_script": {
                    "source": "<script>"
                }
            }
        }
    }
}
```

- <field_name>：指定要搜索的字段名，这个字段通常是一个多值字段。
- terms：提供一组词项，用于指定字段中进行匹配。
- minimum_should_match_field：指定一个包含匹配数量的字段名，其值应用作为要匹配的最少术语数，以便返回文档。
- minimum_should_match_script：提供一个自定义脚本，用于动态计算匹配数量。如果需要动态设置匹配所需的术语数，这个参数非常有用。



**示例：**

一个电影数据库，其中没每部电影都有多个标签。现在，希望找到具有一定数量的给定的标签的电影。

```json
PUT /movies
{
    "mappings": {
        "properties": {
            "title": {
                "type": "text"
            },
            "tag": {
                "type": "keyword"
            },
            "tag_count": {
                "type": "integer"
            }
        }
    }
}

POST /movies/_bulk
{"index": {"_id": 1}}
{"title": "电影1", "tags": ["喜剧", "动作", "科幻"], "tag_count": 3}
{"index": {"_id": 2}}
{"title": "电影3", "tags": ["喜剧", "爱情", "家庭"], "tag_count": 3}
{"index": {"_id": 3}}
{"title": "电影3", "tags": ["动作", "科幻", "家庭"], "tag_count": 3}
```

使用固定数量的term进行匹配

```json
GET /movies/_search
{
    "query": {
        "terms_set": {
            "tags": {
                "terms": ["喜剧", "动作", "科幻"],
                "minimum_should_match": 2
            }
        }
    }
}

GET /movies/_search
{
    "query": {
        "terms_set": {
            "tags": {
                "terms": ["喜剧", "动作", "科幻"],
                "minimum_should_match_script": {
                    "source": "2"
                }
            }
        }
    }
}
```

使用动态计算的term数量进行匹配

```json
GET /movies/_search
{
    "query": {
        "terms_set": {
            "tags": ["喜剧", "动作", "科幻"],
            // 指定固定值为3
            // "minimum_should_match": 3
            // 指定了一个字段tags_count，具体的值是这个字段tags_count存储的值，这里存储的是3
            "minimum_should_match_field": "tags_count"
        }
    }
}

GET /movies/_search
{
    "query": {
        "terms_set": {
            "tags": ["喜剧", "动作", "科幻"],
            // 通过计算而得到的值
            "source": "doc['tag_count'].value*0.7"
        }
    }
}
```



### ▲全文检索

全文检索指在基于相关性搜索和匹配文本数据。这些查询会对输入的文本进行分析，将其拆分为词项（单个单词），并执行诸如分词、词干处理和标准化等操作。

此类检索主要应用于非结构化文本数据，如文章和评论等。



#### ▲match分词查询

match查询是一种全文搜索查询，它使用分析器将查询字符串分解成单独的词条，并在倒排索引中搜索这些词条。match查询适用于文本字段，并且可以通过多种参数来调整搜索行为。



match查询，底层逻辑概述：

1、分词：首先，输入的查询文本会被分词器进行分词。分词器会讲文本拆分成一个个词项（terms），如单词、短语或特定字符。

2、匹配计算：一旦查询被分词，ES将根据查询的类型和参数计算文档与查询的匹配度。对于match查询，ES将比较查询的词项与倒排索引中的词项，并计算文档的相关性得分。相关性得分衡量了文档与查询的匹配程度。

3、结果返回：根据相关性得分，ES将返回最匹配的文档作为搜索结果。搜索结果通常按照相关性得分进行拍下，以便最相关的文档排在前面。



**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "match": {
            "<field_name>": "<query_string>"
        }
    }
}

// 分词后or的效果
GET /employee/_search
{
    "query": {
        "match": {
            "address": "广州白云山公园"
        }
    }
}

// 分词后and的效果
GET /employee/_search
{
    "query": {
        "match": {
            "address": {
                "query": "广州白云山公园",
                "operator": "and"
            }
        }
    }
}
```

- <index_name> 搜索的索引名称
- <field_name> 搜索的字段名称
- <query_string> 要搜索的文本字符串



在match中的应用，当operator参数设置为or时，minimum_should_match参数用来控制匹配的分词的最少数量。

```json
GET /employee/_search
{
    "query": {
        "match": {
            "address": {
                "query": "广州白云山公园",
                "minimum_should_match": 2
            }
        }
    }
}
```



#### ▲multi_match多字段查询

用于在多个字段上执行相同的搜索操作。可以接受一个查询字符串，并在指定的字段集合中搜索这个字符串。multi_match查询提供了灵活的匹配规则和操作符选项，以便根据不同的搜索需求调整搜索行为。



**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "multi_match": {
            "query": "<query_string>",
            "fields": ["<field1>", "<field2>", ...]
        }
    }
}
```

- <index_name> 搜索的索引名称
- <query_string> 要在多个字段中搜索的字符串
- <field1>,<field2>,... 要搜索的字符串列表

```json
GET /employee/_search
{
    "query": {
        "multi_match": {
            "query": "长沙java",
            "fields": ["address", "remark"]
        }
    }
}
// address like '%长沙java%' or remark like '%长沙java%'
```

`multi_match` 会在多个字段里查找匹配项，只要在任意字段中匹配到就可能返回文档。





#### match_phrase短语查询

查询在ES中用于执行短语搜索，它不仅匹配整个短语，**而且还考虑了短语中各个词的顺序和位置**。对于搜索精确短语非常有用，尤其是在用户输入的查询与文档中的文本表达式需要严格匹配时。

**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "match_phrase": {
            "<field_name>": {
                "query": "<phrase>"
            }
        }
    }
}
```

match_phrase查询还支持一个可选字段slop参数，用于指定短语中词之间可以出现的最大位移数量。默认值为0，意味着短语中的词必须严格按照顺序出现。如果设置了非零的slop值，则允许短语中的某些词在一定范围内错位。



示例

```json
GET /employee/_search
{
    "query": {
        "match_phrase": {
            "address": "广州白云山"
        }
    }
}

GET /employee/_search
{
    "query": {
        "match_phrase": {
            "address": "广州白云"
        }
    }
}
```

思考：为什么查询广州白云山有数据，广州白云没有数据？

分析原因：

先查看广州白云山公园的分词结果，可以知道广州和白云不是相邻词条，中间会隔一个白云山，而match_phrase匹配的时相邻的词条，所以查询"广州白云山"有结果，但查询"广州白云"没有结果。

**analysis-ik分词器**

- ik_max_word：这是一种最大词长分词模式。它会将文本做最细粒度的拆分，尝试找出所有可能的词，分词结果会比较多。适合在对文本进行全文搜索，需要尽可能全面地匹配关键词的场景中使用。
- ik_smart：这是一种智能分词模式。它会做最粗粒度的拆分，尽可能地将文本拆分成比较大的词块，分词结果相对较少且更符合语义上的理解。适合对文本进行快速的语义分析，在需要对搜索结果进行宽泛匹配的场景中表现较好。

```json
// ik_max_word
POST _analyze
{
  "analyzer": "ik_max_word",
  "text": "广州白云山"
}

// ik_smart
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "广州白云山"
}
```

**analysis-icu分词器**

```json
POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "广州白云山公园"
}
```

**analysis-smartcn分词器**

```json
POST _analyze
{
  "analyzer": "smartcn",
  "text": "广州白云山公园"
}
```



如何解决词条间隔的问题：借助slot参数，slop参数告知match_phrase查询词条能够相隔多远时任然将文档视为匹配。

```json
GET /employee/_search
{
    "query": {
        "match_phrase": {
            "address": {
                "query": "广州白云",
                "slot": 2
            }
        }
    }
}
```



#### query_string支持与或非表达式的查询

支持灵活的查询类型，允许使用Luence查询语法来构建复杂的搜索查询。这种查询类型支持多种逻辑运算符，包括与（AND）、或（OR）和非（NOT）,以通配符、模糊搜索和正则表达式功能。query_string查询可以在单个或者多个字段上进行搜索，并且可以处理复杂的查询逻辑。

场景：应用场景包括高级搜索、数据分析和报表等，适合处理需要满足特定需求、要求支持与或非表达式的复杂查询任务，通常用于专业领域或需要高级查询功能的应用中。

**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "query_string": {
            "query": "<your_query_string>",
            "default_field": "<field_name>"
        }
    }
}
```

- <your_query_string> 查询逻辑，可以包含shagnshu提到的逻辑运算符和通配符。
- <field_name> 默认搜索字段，如果省略则会搜索所有可索引的字段。

```json
// 未指定查询字段
// AND要求大写
GET /employee/_search
{
    "query": {
        "query_string": {
            "query": "赵六 AND 橘子洲"
        }
    }
}

// 指定单个字段查询
GET /employee/_search
{
    "query": {
        "query_string": {
            "default_field": "address",
            "query": "白云山 OR 橘子洲"
        }
    }
}

// 指定多个字段查询
GET /employee/_search
{
    "query": {
        "query_string": {
            "default_field": ["name", "address"],
            "query": "张三 OR (广州 AND 王五)"
        }
    }
}
```

注意：查询字段分词，则将查询条件分词查询。查询字段不分词，则将查询条件不分词查询。



#### simple_query_string

类似Query String， 但是会忽略错误的语法，同时只支持部分查询语法，不支持AND OR NOT，会当做字符串处理。支持部分逻辑：

- +替代AND
- |替代OR
- -替代NOT

**在生产环境体检使用simple_query_string而不是query_string**主要是因为：simple_query_string提供了宽松的语法，能够容忍一定程度的输入错误，而不导致整个查询失败。

**基本语法**

```json
GET /<index_name>/_search
{
    "query": {
        "simple_query_string": {
            "query": "<query_string>",
            "fields": ["<field1>", "<field2>", ...],
            "default_operator": "OR" // 或者 "AND"
        }
    }
}
```

- default_operator定义了查询字符串中未指定操作符时的默认逻辑运算符，默认是OR，可以是"OR"或"AND"。

```json
// 默认operator是OR
GET /employee/_search
{
    "query": {
        "simple_query_string": {
            // 如果字段分词了，这里查询也会分词，即为 广州、公园分词
            "query": "广州公园",
            "fields": ["name", "address",],
            "default_operator": "AND"
        }
    }
}

// 跟上面效果相同
GET /employee/_search
{
    "query": {
        "simple_query_string": {
            "query": "广州 + 公园",
            // "query": "广州 + 公园 AND", // 语法错误，会忽略，不报错，返回数据
            "fields": ["name", "address",]
        }
    }
}
```



#### 精确匹配与全网检索区别

- 精确不对检索文本进行分词处理，而是将整个文本视为一个完整的词条进行匹配
- 全网检索则需要对文本进行分词处理。在分词后，每个词条将单独进行检索，并通过布尔逻辑（如与、或、非等）进行组合检索，以找到最相关的结果。



### ▲组合查询

#### ▲bool query布尔查询

布尔查询可以安装布尔逻辑条件组织多条查询语句，只有符合整个布尔条件的文档才会被搜索出来。

在布尔条件中，可以包含两种不同的上下文。

- **搜索上下文（query context）**：使用搜索上下文时，ES需要计算每个文档与搜索条件的相关度得分，这个得分的计算需要使用一套复杂的计算公式，**有一定的性能开销，带文本分析的全文检索的查询语句很适合放在搜索上下文中。**
- **过滤上下文（filter context）**：使用过滤上下文时，ES值需要判断搜索条件跟文档数据是否匹配，例如使用Term query判断一个值是否跟搜索内容一致，使用Range query判断某个数据是否位于某个区间等。**过滤上下文的查询不需要进行关联度得分计算，还可以使用缓存加快响应速度，很多术语级查询语句都适合放在过滤上下文中。**

布尔查询一共支持4中组合类型：

| 类型     | 上下文     | 说明                                                         |
| -------- | ---------- | ------------------------------------------------------------ |
| must     | 搜索上下文 | 可包含多个查询条件，每个条件均满足的文档才能被搜索到，每次查询需要计算相关度得分，属于搜索上下文 |
| should   | 搜索上下文 | 可包含多个查询条件，不存在must和filter条件时，至少需要满足多个查询条件中的一个，文档才能被搜索到，否则需要满足的条件数量不受限制，匹配到的查询越多相关度越高，也术语搜索上下文 |
| filter   | 过滤上下文 | 可包含多个过滤条件，每个条件均满足的文档才能被搜索到，每个过滤条件不计算相关度得分，结果在一定条件下被缓存，属于过滤上下文 |
| must_not | 过滤上下文 | 可包含多个过滤条件，每个条件均不满足的文档才会被搜索到，每个过滤条件不计算相关度得分，结果在一定条件下会被缓存，属于过滤上下文 |

```json
PUT /books
{
    "settings": {
        "number_of_replicas": 1,
        "number_of_shards": 1
    },
    "mappings": {
        "properties": {
            "id": {
                "type": "long"
            },
            "title": {
                "type": "text",
                "analyzer": "ik_max_word"
            },
            "language": {
                "type": "keyword"
            },
            "author": {
                "type": "keyword"
            },
            "price": {
                "type": "double"
            },
            "publish_time": {
                "type": "date",
                "format": "yyyy-MM-dd"
            },
            "descirption": {
                "type": "text",
                "analyzer": "ik_max_word"
            }
        }
    }
}

PUT /_bulk
{"index": {"_index": "books", "_id": "1"}}
{"id": "1", "title": "Java编程思想", "language": "java", "author": "Bruce Eckel", "price": 70.20, "publish_time": "2007-10-01", "description": "Java学习必读经典，殿堂级著作！赢得了全球程序员的广泛赞誉。"}
{"index": {"_index": "books", "_id": "2"}}
{"id": "2", "title": "Java程序性能优化", "language": "java", "author": "葛一鸣", "price": 46.5, "publish_time": "2012-08-01", "description": "让你的Java程序更快、更稳定。深入剖析软件设计层面、代码层面、JVM虚拟机层面的优化方法"}
{"index": {"_index": "books", "_id": "3"}}
{"id": "3", "title": "Python科学计算", "language": "python", "author": "张若愚", "price": 81.4, "publish_time": "2016-05-01", "description": "零基础学Python，光盘中作者独家整合开发win Python运行环境，涵盖了Python各个扩张库"}
{"index": {"_index": "books", "_id": "4"}}
{"id": "4", "title": "Python基础教程", "language": "python", "author": "Helant", "price": 51.50, "publish_time": "2014-03-01", "description": "经典的Python入门教程，层次鲜明，结构严谨，内容详实"}
{"index": {"_index": "books", "_id": "5"}}
{"id": "5", "title": "JavaScript高级程序设计", "language": "javascript", "author": "Nicholas C. Zakas", "price": 66.4, "publish_time": "2012-10-01", "description": "JavaScript技术经典名著"}

GET /books/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        // title文本类型，并且分词：所以搜索 “Java” 或者 “编程” 满足条件的就能返回
                        "title": "Java编程"
                    }
                },
                {
                    "match": {
                        "description": "性能优化"
                    }
                }
            ]
        }
    }
}

GET /books/_search
{
    "query": {
        "bool": {
            "should": [
                {
                    "match": {
                        "title": "Java编程"
                    }
                },
                {
                    "match": {
                        "description": "性能优化"
                    }
                }
            ],
            "minimum_should_match": 1
        }
    }
}

GET /books/_search
{
    "query": {
        "bool": {
            "filter": [
                {
                    // 精确匹配，没必要算分，使用filter
                    "term": {
                        "language": "java"
                    }
                },
                {
                    "range": {
                        "publish_time": {
                            "gte": "2010-08-01"
                        }
                    }
                }
            ]
        }
    }
}
```

多个条件查询还可以使用以下方式，但是意义不同

```json
// title like '%Java编程%' AND description like '%性能优化%'
GET /books/_search
{
    "query": {
        "bool": {
            "must": [
                {"match": {"title": "Java编程"}},
                {"match": {"description": "性能优化"}}
            ]
        }
    }
}

// 使用multi_match表示多个条件
// title like '%Java编程%' or title like '%性能优化%' or description like '%Java编程%' or description like '%性能优化%'
GET /books/_search
{
    "query": {
        "multi_match": {
            "query": "Java编程 性能优化",
            "fields": ["title", "description"]
        }
    }
}

// simple_query_string 查询使用 AND 操作符来要求 title 字段必须包含 “Java 编程” 且 description 字段必须包含 “性能优化”，这和原 bool 查询的语义是一致的。
GET /books/_search
{
    "query": {
        "simple_query_string": {
            "query": "title:Java编程 AND description:性能优化",
            "fields": ["title", "description"]
        }
    }
}
```



### highlight高亮显示实现

highlight关键字：可以让符合条件的文档中的关键字高亮。

highlight相关属性：

- pre_tags 前缀标签
- post_tags 后缀标签
- tags_schema 设置为style可以使用内置高亮样式
- require_field_match 多字段高亮需要设置为false

```json
// 指定ik分词器
PUT /products
{
    "settings": {
        "index": {
            "analysis.analyzer.default.type": "ik_max_word"
        }
    }
}

PUT /products/_doc/1
{
    "proId": "2",
    "name": "牛仔男外套",
    "desc": "牛仔外套男装春季衣服男春季夹克修身休闲男生潮牌工装潮流头号亲年春秋棒球服男 7705浅蓝常规 XL",
    "timestamp": 1576313264451,
    "createTime": "2019-12-13 12:56:56"
}

PUT /products/_doc/2
{
    "proId": "6",
    "name": "HLA海澜之家牛仔裤男",
    "desc": "HLA海澜之家牛仔裤男2019时尚有型舒适HKNAD3E109A 牛仔蓝(A9)175/82A(32)",
    "timestamp": 1576314265571,
    "createTime": "2019-12-18 15:56:56"
}

GET /products/_search
{
    "query": {
        "term": {
            "name": {
                "value": "牛仔"
            }
        }
    },
    "highlight": {
        "fields": {
            // *表示前面用到的字段，这里是term中的name字段
            // 不指定是全部字段
            // 可以指定字段，则只对添加这里指定的字段添加标签
            "*": {}
        }
    }
}
```

**自定义高亮html标签**

可以在highlight中使用pre_tags和post_tags

```json
GET /products/_search
{
    "query": {
        "multi_match": {
            "fields": ["name", "desc"],
            "query": "牛仔"
        }
    },
    "highlight": {
        "pre_tags": ["<span style='colorLred'>"],
        "post_tags": ["</span>"],
        "fields": {
            // *表示前面用到的字段，这里是term中的name和desc字段
            "*": {}
        }
    }
}
```

**多字段高亮**

```json
GET /products/_search
{
    "query": {
        "term": {
            "name": {
                "value": "牛仔"
            }
        }
    },
    "highlight": {
        "pre_tags": ["<font color='red'>"],
        "post_tags": ["</font>"],
        // 多字段高亮需要设置为false
        "require_field_match": "false",
        "fields": {
            // 指定具体哪些字段高亮
            "name": {},
            "desc": {}
        }
    }
}
```



### 地理空间位置查询

地理空间位置查询是数据库和我搜索系统中的一个重要特性，特别是在地理信息系统(GIS)和位置服务中。它允许用户基于地理位置信息来搜索和过滤数据。在ES这样的全文搜索引擎中，地理空间位置查询被广泛应用，例如在旅行、房地产、物流和零售等行业，用于提供基础位置和搜索功能。

在ES中，地理空间数据通常存储在geo_point字段类型中。这种字段类型可以存储维度和精度坐标，用于表示地球上的一个点。

以下是一个使用geo_distance查询的例子，它会找到距离特定点一定距离内的所有文档。

1、确保索引中有一个geo_point字段，例如location。

```json
PUT /index-geo
{
    "mappings": {
        "properties": {
          "name": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_max_word"
          },
           "description": {
            "type": "text"
          },
          "location": {
            "type": "geo_point"
          }
        }
    }
}

POST /index-geo/_bulk
{ "index": { "_index": "index-geo" } }
{ "name": "示例地点1", "description": "批量插入测试1", "location": { "lat": 40, "lon": 116.4 } }
{ "index": { "_index": "index-geo" } }
{ "name": "示例地点2", "description": "批量插入测试2", "location": { "lat": 39.9, "lon": 116.4 } }
{ "index": { "_index": "index-geo" } }
{ "name": "示例地点3", "description": "批量插入测试3", "location": { "lat": 39.9, "lon": 116.4010 } }
```

2、使用以下查询来找到距离给定坐标点（例如lat和lon）小于或等于10公里的所有文档：

```json
GET /index-geo/_search
{
    "query": {
        "bool": {
            "must": {
                "match_all": {}
            },
            "filter": {
                "geo_distance": {
                    "distance": "10km",
                    "distance_type": "arc",
                    "location": {
                        "lat": 39.9,
                        "lon": 116.4
                    }
                }
            }
        }
    }
}
```

- bool	逻辑查询容器，用于组合多个查询子句
- match_all 匹配所有文档的查询子句
- geo_distance 地理距离查询，允许指定一个距离和点的坐标
- distance_type 可以是arc（以地球表面的弧度单位）或plane（以直线距离为单位）。通常对于地球上的距离查询，建议使用arc
- location 查询的参考点，不包含维度和经度坐标





### ElasticSearch8.x向量检索

ES 8.x引入了一个重要的特性：向量检索（Vector Search）,特别是通过KNN（K-Nearest Neighbors）算法支持向量近邻检索。这一特性使得ES在机器学习、数据分析和推荐系统等领域的应用变得更加广泛和强大。

向量检索的基本思路是，将文档（或数据项）表示为高维向量，并使用这些向量来执行相似性搜索。在ES中，这些向量被存储在dense_vector类型的字段中，然后使用KNN算法来找到与给定向量最相似的其他向量。

```json
PUT /image-index
{
    "mappings": {
        "properties": {
            "image-vector": {
                "type": "dense_vector",
	            "dims": 3
            },
            "title": {
                "type": "text"
            },
            "file-type": {
                "type": "keywrod"
            },
            "my_label": {
                "type": "text"
            }
        }
    }
}

POST image-index/_bulk
{"index"： {}}
{"image-vector": [-5, 9, 12], "title": "Image A", "file-type": "jpeg", "my_label": "red"}
{"index"： {}}
{"image-vector": [10, -2, 4], "title": "Image B", "file-type": "png", "my_label": "blue"}
{"index"： {}}
{"image-vector": [4, 0, -1], "title": "Image C", "file-type": "gif", "my_label": "red"}

POST image-index/_search
{
    "knn": {
        "field": "image-vector",
        "query_vector": [-4, 10, -12],
        "k": 10,
        "num_candidates": 100
    },
    "fields": ["title", "file-type"]
}
```



## 搜索相关性评分

### 相关性概述

在搜索引擎中，描述一个文档与查询语句匹配程度的度量标准。

| 关键词   | 文档ID      |
| -------- | ----------- |
| JAVA     | 1,2,3       |
| 设计模式 | 1,2,3,4,5,6 |
| 多线程   | 2,3,7,9     |

ES使用评分算法，根据查询条件与搜索索引文档的匹配程度来确定每个文档的相关性。

如果同时查询 JAVA/设计模式/多线程 ：2，3文档的评分更高。

查询结果中的：_score就是ES检索后返回的评分，文档评分越高，则该文档的相关性越高。



### 计算相关性评分

ES 5之前的版本，评分机制或者打分模型是基于TF-IDF实现的。从ES 5之后，默认的打分机制改为了Okapi BM25

#### TF-IDF算法

3个决定性因素指标：

- TF 词频：检索词(分词后)在**文档中**出现的频率越高，相关性也越高。
- IDF 逆向文本频率：每个检索词**在索引中出现**的频率，频率越高，相关度**越低**。
- 字段长度归与一类：检索词出现在一个内容短的title要比同样的词出现在一个内容长的content字段的权重更大。



##### **TF算分说明**

搜索 “小米旗舰手机”

**ES文档存储示例**

| id   | goods_name                   | describe                | 匹配数 |
| ---- | ---------------------------- | ----------------------- | ------ |
| 1    | **小米** **手机**            | 手机中的战斗机          | 2      |
| 2    | **小米**NFC**旗舰** **手机** | 小米手机，支持全功能NFC | 3      |
| 3    | **旗舰**NFC**手机**          |                         | 2      |
| 4    | **小米**耳机                 |                         | 1      |
| 5    | 红米耳机                     |                         |        |
| 6    | 红米手机                     |                         |        |

**ES索引示例**

| 是否命中索引 | Term Dic（分词、倒排索引） | Posting List（索引对应的id） |
| ------------ | -------------------------- | ---------------------------- |
| ✅            | 小米                       | 1,2,4                        |
| ✅            | 旗舰                       | 2,3                          |
| ✅            | 手机                       | 1,2,3                        |
|              | 耳机                       | 4,5                          |
|              | 红米                       | 5,6                          |
|              | NFC                        | 2,3                          |

评分高的是id=2的文档，在索引中出现了3次



##### BM25

和经典的TF-IDF相比，当TF无限增加时，BM25算分会趋于一个数值。



##### 查看分值Explain

通过Explain API查看TF-IDF

```json
GET /test_score/_search
{
    "explain": true,
    "query": {
        "match": {
            "content": "elasticsearch"
        }
    }
}

GET /test_score/_explain/2
{
    "query": {
        "match": {
            "content": "elasticsearch"
        }
    }
}
```



### 自定义评分策略

#### Index Boost: 在索引层面修改相关性

#### boosting: 修改文档相关性

#### negative_boost: 降低相关性

#### function_score: 自定义评分

#### rescore_query: 查询后二次打分



### 多字段搜索场景优化



## 运营维护

### 插件

插件首页

https://www.elastic.co/guide/en/elasticsearch/plugins/7.17/index.html

https://www.elastic.co/guide/en/elasticsearch/plugins/7.17/intro.html

分词插件

https://www.elastic.co/guide/en/elasticsearch/plugins/7.17/analysis.html



#### 自动安装

```shell
# 查看已安装的插件
bin/elasticsearch-plugin list

# 安装插件
bin/elasticsearch-plugin install <plugin-name-or-url>
bin/elasticsearch-plugin install https://some.domain/path/to/plugin.zip
bin/elasticsearch-plugin install file:///path/to/plugin.zip
bin\elasticsearch-plugin install file:///C:/path/to/plugin.zip
bin/elasticsearch-plugin install analysis-icu repository-s3

# ICU分词插件
bin/elasticsearch-plugin install analysis-icu
# 中文分词插件
bin/elasticsearch-plugin install analysis-smartcn
bin/elasticsearch-plugin install https://get.infini.cloud/elasticsearch/analysis-ik/8.4.1
bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.10.0/elasticsearch-analysis-ik-8.10.0.zip


# 删除插件
bin/elasticsearch-plugin remove <plugin-name>
bin/elasticsearch-plugin remove analysis-icu

# 重启服务
sudo systemctl restart elasticsearch

# 验证插件安装
http://localhost:9200/_cat/plugins?v
或者
curl -X GET "localhost:9200/_cat/plugins?v"

name         component    version
7c780da0208b analysis-icu 7.17.5
7c780da0208b analysis-ik  7.17.5
```



#### 手动安装

1. 下载插件的 ZIP 文件。
2. 将 ZIP 文件放到 Elasticsearch 的 `plugins` 目录下。
3. 解压 ZIP 文件。
4. 重启 Elasticsearch。

```shell
# /usr/share/elasticsearch/plugins

mkdir -p plugins/my-plugin
unzip my-plugin.zip -d plugins/my-plugin

# 重启服务
sudo systemctl restart elasticsearch
```



#### 常见插件

以下是一些常见的 Elasticsearch 插件：

- **Analysis Plugins**: 提供额外的文本分析功能。
  - `analysis-icu`: 提供 ICU 分析器。
  - `analysis-kuromoji`: 提供日文分析器。
  - `analysis-smartcn`: 提供中文分析器。
- **Discovery Plugins**: 提供集群节点发现机制。
  - `discovery-ec2`: 用于在 AWS EC2 环境中发现节点。
- **Security Plugins**: 提供安全功能。
  - `x-pack`: 提供安全、监控、报警等功能。
- **Monitoring Plugins**: 提供监控功能。
  - `x-pack`: 提供监控和管理功能。



## 参考

[【2024版ES8.x教程】这可能是B站唯一能将ElasticSearch8.x讲明白的教程](https://www.bilibili.com/video/BV1TjpcehEti)

[ElasticSearch中字符串keyword和text类型区别](https://baijiahao.baidu.com/s?id=1763386735571186685)

