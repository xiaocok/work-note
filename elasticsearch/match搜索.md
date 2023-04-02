
想搜索特定的字段，可以使用匹配查询。如下所示，请求搜索地址字段以查询地址中包含 mill 或 lane 的客户

```text
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```

返回结果
```json
{
  "took" : 16,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 19,
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
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "970",
        "_score" : 5.4032025,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,
          "firstname" : "Forbes",
          "lastname" : "Wallace",
          "age" : 28,
          "gender" : "M",
          "address" : "990 Mill Road",
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "345",
        "_score" : 5.4032025,
        "_source" : {
          "account_number" : 345,
          "balance" : 9812,
          "firstname" : "Parker",
          "lastname" : "Hines",
          "age" : 38,
          "gender" : "M",
          "address" : "715 Mill Avenue",
          "employer" : "Baluba",
          "email" : "parkerhines@baluba.com",
          "city" : "Blackgum",
          "state" : "KY"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "472",
        "_score" : 5.4032025,
        "_source" : {
          "account_number" : 472,
          "balance" : 25571,
          "firstname" : "Lee",
          "lastname" : "Long",
          "age" : 32,
          "gender" : "F",
          "address" : "288 Mill Street",
          "employer" : "Comverges",
          "email" : "leelong@comverges.com",
          "city" : "Movico",
          "state" : "MT"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 4.1042743,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "70",
        "_score" : 4.1042743,
        "_source" : {
          "account_number" : 70,
          "balance" : 38172,
          "firstname" : "Deidre",
          "lastname" : "Thompson",
          "age" : 33,
          "gender" : "F",
          "address" : "685 School Lane",
          "employer" : "Netplode",
          "email" : "deidrethompson@netplode.com",
          "city" : "Chestnut",
          "state" : "GA"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "556",
        "_score" : 4.1042743,
        "_source" : {
          "account_number" : 556,
          "balance" : 36420,
          "firstname" : "Collier",
          "lastname" : "Odonnell",
          "age" : 35,
          "gender" : "M",
          "address" : "591 Nolans Lane",
          "employer" : "Sultraxin",
          "email" : "collierodonnell@sultraxin.com",
          "city" : "Fulford",
          "state" : "MD"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "568",
        "_score" : 4.1042743,
        "_source" : {
          "account_number" : 568,
          "balance" : 36628,
          "firstname" : "Lesa",
          "lastname" : "Maynard",
          "age" : 29,
          "gender" : "F",
          "address" : "295 Whitty Lane",
          "employer" : "Coash",
          "email" : "lesamaynard@coash.com",
          "city" : "Broadlands",
          "state" : "VT"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "715",
        "_score" : 4.1042743,
        "_source" : {
          "account_number" : 715,
          "balance" : 23734,
          "firstname" : "Tammi",
          "lastname" : "Hodge",
          "age" : 24,
          "gender" : "M",
          "address" : "865 Church Lane",
          "employer" : "Netur",
          "email" : "tammihodge@netur.com",
          "city" : "Lacomb",
          "state" : "KS"
        }
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "449",
        "_score" : 4.1042743,
        "_source" : {
          "account_number" : 449,
          "balance" : 41950,
          "firstname" : "Barnett",
          "lastname" : "Cantrell",
          "age" : 39,
          "gender" : "F",
          "address" : "945 Bedell Lane",
          "employer" : "Zentility",
          "email" : "barnettcantrell@zentility.com",
          "city" : "Swartzville",
          "state" : "ND"
        }
      }
    ]
  }
}
```