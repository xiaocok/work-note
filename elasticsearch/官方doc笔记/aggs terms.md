
## 银行例子
请求使用 terms 聚合对 bank 索引中对所有还在账户进行分组，并安装返回前 10 个账户数最多的州：
```text
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

这里为啥是state.keyword，而不是state，通过_mapping查询表结构，可以看出state的字段的fields中有一个keyword属性。
一般是字符串类型的数据，都有一个keyword的属性，查询时，都需要给定这个keyword进行查询。
```text
GET /bank/_mapping
返回结果
{
  "bank" : {
    "mappings" : {
      "properties" : {
        "account_number" : {
          "type" : "long"
        },
        "address" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "age" : {
          "type" : "long"
        },
        "balance" : {
          "type" : "long"
        },
        ......
        "state" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```


返回结果
```json
{
  "took" : 40,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30
        },
        {
          "key" : "MD",
          "doc_count" : 28
        },
        {
          "key" : "ID",
          "doc_count" : 27
        },
        {
          "key" : "AL",
          "doc_count" : 25
        },
        {
          "key" : "ME",
          "doc_count" : 25
        },
        {
          "key" : "TN",
          "doc_count" : 25
        },
        {
          "key" : "WY",
          "doc_count" : 25
        },
        {
          "key" : "DC",
          "doc_count" : 24
        },
        {
          "key" : "MA",
          "doc_count" : 24
        },
        {
          "key" : "ND",
          "doc_count" : 24
        }
      ]
    }
  }
}
```
如图所示：在返回结果中buckets 的值是字段 state 的桶。doc_count 表示每州的账户数。你可以看到爱达荷州中有 27 个帐户，由于 "size": 0 表示了返回只有聚合结果，无具体数据。


你也组合聚合以构建更为复杂的统计数据。如图所示：请求将 avg 聚合嵌套在 group_by_state 聚合内，以计算每州的平均账户余额。
```text
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

返回结果
```json
{
  "took" : 22,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30,
          "average_balance" : {
            "value" : 26073.3
          }
        },
        {
          "key" : "MD",
          "doc_count" : 28,
          "average_balance" : {
            "value" : 26161.535714285714
          }
        },
        {
          "key" : "ID",
          "doc_count" : 27,
          "average_balance" : {
            "value" : 24368.777777777777
          }
        },
        {
          "key" : "AL",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 25739.56
          }
        },
        {
          "key" : "ME",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 21663.0
          }
        },
        {
          "key" : "TN",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 28365.4
          }
        },
        {
          "key" : "WY",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 21731.52
          }
        },
        {
          "key" : "DC",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 23180.583333333332
          }
        },
        {
          "key" : "MA",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 29600.333333333332
          }
        },
        {
          "key" : "ND",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 26577.333333333332
          }
        }
      ]
    }
  }
}
```


可以不按内置结果进行排序，而可以使用聚合后的 average_balance 字段进行排序
```text
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

返回结果
```json
{
  "took" : 14,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : -1,
      "sum_other_doc_count" : 827,
      "buckets" : [
        {
          "key" : "CO",
          "doc_count" : 14,
          "average_balance" : {
            "value" : 32460.35714285714
          }
        },
        {
          "key" : "NE",
          "doc_count" : 16,
          "average_balance" : {
            "value" : 32041.5625
          }
        },
        {
          "key" : "AZ",
          "doc_count" : 14,
          "average_balance" : {
            "value" : 31634.785714285714
          }
        },
        {
          "key" : "MT",
          "doc_count" : 17,
          "average_balance" : {
            "value" : 31147.41176470588
          }
        },
        {
          "key" : "VA",
          "doc_count" : 16,
          "average_balance" : {
            "value" : 30600.0625
          }
        },
        {
          "key" : "GA",
          "doc_count" : 19,
          "average_balance" : {
            "value" : 30089.0
          }
        },
        {
          "key" : "MA",
          "doc_count" : 24,
          "average_balance" : {
            "value" : 29600.333333333332
          }
        },
        {
          "key" : "IL",
          "doc_count" : 22,
          "average_balance" : {
            "value" : 29489.727272727272
          }
        },
        {
          "key" : "NM",
          "doc_count" : 14,
          "average_balance" : {
            "value" : 28792.64285714286
          }
        },
        {
          "key" : "LA",
          "doc_count" : 17,
          "average_balance" : {
            "value" : 28791.823529411766
          }
        }
      ]
    }
  }
}
```

除了这些基本的桶跟指标聚合之外，Elasticsearch 还提供了其他类型的聚合，用于对多个字段进行操作和分析特定类型的数据，如日期、IP 和地理数据。你还可以将单个聚合的结果送到管道聚合中进行深一步分析。

聚合提供了核心分析功能支持机器学习检测异常等高级功能。



## 汽车交易的信息
创建一些对汽车经销商有用的聚合，数据是关于汽车交易的信息：车型、制造商、售价、何时被出售等。

首先我们批量索引一些数据：
```text
POST /cars/transactions/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }
```

有了数据，开始构建我们的第一个聚合。汽车经销商可能会想知道哪个颜色的汽车销量最好，用聚合可以轻易得到结果，用 terms 桶操作：
```text
GET /cars/transactions/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color.keyword"
            }
        }
    }
}
```

* size：可能会注意到我们将 size 设置成 0 。我们并不关心搜索结果的具体内容，所以将返回记录数设置为 0 来提高查询速度。 设置 size: 0 与 Elasticsearch 1.x 中使用 count 搜索类型等价。
* aggs：聚合操作被置于顶层参数 aggs 之下（如果你愿意，完整形式 aggregations 同样有效）。
* popular_colors：然后，可以为聚合指定一个我们想要名称，本例中是： popular_colors 。
* terms：最后，定义单个桶的类型 terms 。

然后我们为聚合定义一个名字，名字的选择取决于使用者，响应的结果会以我们定义的名字为标签，这样应用就可以解析得到的结果。

随后我们定义聚合本身，在本例中，我们定义了一个单 terms 桶。 这个 terms 桶会为每个碰到的唯一词项动态创建新的桶。 因为我们告诉它使用 color 字段，所以 terms 桶会为每个颜色动态创建新桶。



## 参考
* [聚合查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_aggregation_test_drive.html)