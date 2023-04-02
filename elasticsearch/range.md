
请求搜索余额在 20000 ~ 30000（包括 30000） 之间的账户
```text
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

返回结果
```text
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 217,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "49",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 49,
          "balance" : 29104,
          "firstname" : "Fulton",
          "lastname" : "Holt",
          "age" : 23,
          "gender" : "F",
          "address" : "451 Humboldt Street",
          "employer" : "Anocha",
          "email" : "fultonholt@anocha.com",
          "city" : "Sunriver",
          "state" : "RI"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "102",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 102,
          "balance" : 29712,
          "firstname" : "Dena",
          "lastname" : "Olson",
          "age" : 27,
          "gender" : "F",
          "address" : "759 Newkirk Avenue",
          "employer" : "Hinway",
          "email" : "denaolson@hinway.com",
          "city" : "Choctaw",
          "state" : "NJ"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "133",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 133,
          "balance" : 26135,
          "firstname" : "Deena",
          "lastname" : "Richmond",
          "age" : 36,
          "gender" : "F",
          "address" : "646 Underhill Avenue",
          "employer" : "Sunclipse",
          "email" : "deenarichmond@sunclipse.com",
          "city" : "Austinburg",
          "state" : "SC"
        }
      },
      ......
    ]
  }
}
```
