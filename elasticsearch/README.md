
# 教程

## 文档操作

[PUT 插入数据](./PUT插入数据.md)

[GET 查询数据](./GET查询数据.md)

[bulk 批量导入数据](./批量导入数据.md)

## 索引查询

查询请求：GET /索引/_search

例如：GET /bank/_search

body使用查询语句
```text
GET /bank/_search
{
  "query": { "match_all": {} },
}
```

_cat说明
```text
GET /_cat/nodes     查看所有节点
GET /_cat/health    查看es健康状况
GET /_cat/master    查看主节点
GET /_cat/indices   查看所有索引 ；相当于 MySQL 的 show databases;
```

_search说明
```text
GET /_search                    空搜索
GET /_search                    在所有的索引中搜索所有的类型
GET /gb/_search                 在 gb 索引中搜索所有的类型
GET /gb,us/_search              在 gb 和 us 索引中搜索所有的文档
GET /g*,u*/_search              在任何以 g 或者 u 开头的索引中搜索所有的类型
GET /gb/user/_search            在 gb 索引中搜索 user 类型
GET /gb,us/user,tweet/_search   在 gb 和 us 索引中搜索 user 和 tweet 类型
GET /_all/user,tweet/_search    在所有的索引中搜索 user 和 tweet 类型
GET /_search?timeout=10ms       超时时间，单位默认毫秒，指定 timeout 为 10 或者 10ms（10毫秒），或者 1s（1秒）
GET /_search?size=5&from=10     翻页，从第10条开始，查询5条

GET /_all/tweet/_search?q=tweet:elasticsearch   查询在 tweet 类型中 tweet 字段包含 elasticsearch 单词的所有文档
```


[查询结果说明](./查询结果说明.md)

[sort排序](./sort排序.md)

[分页](./分页.md)


## match查询
### match说明:  
```text
match_all       搜索全部
match           or搜索：满足条件中的一个即可(一个条件中多个关键词时，多个关键词类似于or组合，不是多个条件的情况)
match_phrase    and搜索：必须满足所有条件才行(一个条件中多个关键词的情况，多个关键词类似于and组合，不是多个条件的情况)
```
> match_all示例，参考上面的【sort排序】【分页】

[match搜索 - mysql的or](match搜索.md)

[match_phrase搜索 - mysql的and](./match_phrase搜索.md)

### must must_not should filter
> 每个 must 或者 should 查询子句中的条件都会影响文档的相关得分。得分越高，文档跟搜索条件匹配得越好。默认情况下，Elasticsearch 返回的文档会根据相关性算分倒序排列。

```text
must:       文档必须完全符合所有条件(内部多个条件时，需要同时满足多个)
should:     should下面会带一个以上的条件，文档至少满足一个条件(内部多个条件时，至少满足其中一个)
must_not:   文档必须不匹配所有条件(其实就是must取反)
filter:     作用与must一致，不过must会有评分，而filter仅仅只是过滤，所以性能更高
```

[must match](./must%20match.md)

[must_not match - mysql的not](./must_not%20match.md)

[should - 多个条件满足一个即可](./should.md)


[range - 范围查询](./range.md)

### 聚合分析
类似于mysql的group by, sum, count, having

[aggs terms](./aggs%20terms.md)

### 表结构
[_mapping 表结构](./mapping%20表结构.md)


### 参考
* [Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
* [Elasticsearch中文文档](https://learnku.com/docs/elasticsearch73/7.3)
* [elasticsearch7.x教程必看教你如何快速入门，精心归纳](https://my.oschina.net/u/4403110/blog/4565352)
* [Elasticsearch - runewbie的博客](https://blog.csdn.net/runewbie/category_10068312.html)

### Elasticsearch Clients驱动
* [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html)
* [Elasticsearch-PHP](https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/index.html)
* [Elasticsearch Go Client](https://www.elastic.co/guide/en/elasticsearch/client/go-api/current/index.html)

### 问题
* [ES中 同时使用should和must 导致只有must生效 解决方案](https://snakey.blog.csdn.net/article/details/108450346)

