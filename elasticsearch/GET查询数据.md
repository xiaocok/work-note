
```text
GET /customer/_doc/1
```

返回
```text
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 26,
  "_primary_term" : 4,
  "found" : true,
  "_source" : {
    "name": "John Doe"
  }
}
```

响应体里还提供了有关搜索请求的相关信息：

took – 查询花费时长（毫秒）
timed_out – 请求是否超时
_shards – 搜索了多少分片，成功、失败或者跳过了多个分片（明细）
max_score – 最相关的文档分数
hits.total.value - 找到的文档总数
hits.sort - 文档排序方式 （如没有则按相关性分数排序）
hits._score - 文档的相关性算分 (match_all 没有算分)


