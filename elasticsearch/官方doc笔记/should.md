
请求在银行索引里搜索年龄是 40 的，或者 居住在爱达荷州 (ID) 的客户：
```text
GET /bank/_search
{
  "query": {
    "bool": {
      "should": [
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
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 70,
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
        "_id" : "695",
        "_score" : 3.5945687,
        "_source" : {
          "account_number" : 695,
          "balance" : 36800,
          "firstname" : "Gonzales",
          "lastname" : "Mcfarland",
          "age" : 26,
          "gender" : "F",
          "address" : "647 Louisa Street",
          "employer" : "Songbird",
          "email" : "gonzalesmcfarland@songbird.com",
          "city" : "Crisman",
          "state" : "ID"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "708",
        "_score" : 3.5945687,
        "_source" : {
          "account_number" : 708,
          "balance" : 34002,
          "firstname" : "May",
          "lastname" : "Ortiz",
          "age" : 28,
          "gender" : "F",
          "address" : "244 Chauncey Street",
          "employer" : "Syntac",
          "email" : "mayortiz@syntac.com",
          "city" : "Munjor",
          "state" : "ID"
        }
      },
      ......
    ]
  }
}
```