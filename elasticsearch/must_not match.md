
### 单独 must_not 使用

请求在银行索引里搜索，不居住在爱达荷州 (ID) 的客户：
```text
GET /bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
```

返回结果
```text
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 18,
      "relation" : "eq"
    },
    "max_score" : 9.507477,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "136",
        "_score" : 9.507477,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      },
      ......
    ]
  }
}
```

### must_not 与 must 组合使用
请求在银行索引里搜索年龄是 40 的，但不居住在爱达荷州 (ID) 的客户：
```text
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

返回结果
```text
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 43,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "474",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 474,
          "balance" : 35896,
          "firstname" : "Obrien",
          "lastname" : "Walton",
          "age" : 40,
          "gender" : "F",
          "address" : "192 Ide Court",
          "employer" : "Suremax",
          "email" : "obrienwalton@suremax.com",
          "city" : "Crucible",
          "state" : "UT"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "479",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 479,
          "balance" : 31865,
          "firstname" : "Cameron",
          "lastname" : "Ross",
          "age" : 40,
          "gender" : "M",
          "address" : "904 Bouck Court",
          "employer" : "Telpod",
          "email" : "cameronross@telpod.com",
          "city" : "Nord",
          "state" : "MO"
        }
      },
      ......
    ]
  }
}
```
