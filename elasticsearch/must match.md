
单个must条件
```text
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ]
    }
  }
}
```

多个must条件  
请求在银行索引里搜索年龄是 40 的，同时居住在爱达荷州 (ID) 的客户：
```text
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } },
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

返回结果
```text
{
  "took" : 8,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 4.5945687,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "291",
        "_score" : 4.5945687,
        "_source" : {
          "account_number" : 291,
          "balance" : 19955,
          "firstname" : "Lynn",
          "lastname" : "Pollard",
          "age" : 40,
          "gender" : "F",
          "address" : "685 Pierrepont Street",
          "employer" : "Slambda",
          "email" : "lynnpollard@slambda.com",
          "city" : "Mappsville",
          "state" : "ID"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "664",
        "_score" : 4.5945687,
        "_source" : {
          "account_number" : 664,
          "balance" : 16163,
          "firstname" : "Hart",
          "lastname" : "Mccormick",
          "age" : 40,
          "gender" : "M",
          "address" : "144 Guider Avenue",
          "employer" : "Dyno",
          "email" : "hartmccormick@dyno.com",
          "city" : "Carbonville",
          "state" : "ID"
        }
      }
    ]
  }
}
```