# 组合查询

在实际的使用过程中，搜索条件需要在多个字段上查询多种多样的文本，并且根据一系列的标准来过滤。为了构建类似的高级查询，需要一种能够将多查询组合称单一查询的查询方法。

## bool

> bool 查询可以将多查询组合在一起，成为用户想要的布尔查询。它接收以下参数：
>
> | 操作符  | 描述                     |
> | -------- | --------------------------------------------- |
> | must   | AND 关系，文档 ***必须*** 匹配这些条件才能检索出来 |
> | must_not | NOT 关系，文档 ***必须不*** 匹配这些条件才能检索出来 |
> | should  | OR 关系，如果满足这些语句中的任意语句，将增加 `_score`，否则，无任何影响。它们主要用于修正每个文档的相关性得分 |
> | filter  | ***必须*** 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文件 |

> Q：相关性得分是如何组合的？
>
> A：每一个子查询都独自地计算文档的相关性得分，一旦它们的得分被计算出来，bool 查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。

```http
GET http://localhost:9200/superz/_search
{
	"query":{
        "bool": {
            "must":     { "match": { "title": "how to make millions" }},
            "must_not": { "match": { "tag":   "spam" }},
            "should": [
                { "match": { "tag": "starred" }},
                { "range": { "date": { "gte": "2014-01-01" }}}
            ],
            "filter":{
            	"range":{"price":{"lte":29.99}}
            }
        }
    }
}
```

> **TIP**：如果没有 must 语句，那么至少需要能够匹配其中的一条 should 语句。但如果存在至少一条 must 语句，则对 should 语句的匹配没有要求。

## constant_score

> constant_score 查询时将一个不变的常量评分应用于所有匹配的文档。它被经常用于只需要执行一个 filter 而没有其他查询的情况下。
>
> 可以使用它来取代只有 filter 语句的 bool 查询。在性能上时完全相同的，但对于提高查询简洁性和清晰度有很大的帮助。

```http
GET http://localhost:9200/superz/_search
{
	"query":{
		"constant_score":{
			"filter":{
				"term":{ "name":"superz1" }
			}
		}
	}
}
```



