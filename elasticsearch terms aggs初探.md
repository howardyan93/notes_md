elasticsearch terms aggs初探
===================================
通过elasticsearch的aggs，就可以方便的对数据进行初步的统计。比如结合terms的bucket。就可以统计出某个field的所有出现过的type，类似于mongodb的distinct。但是如果此字段不是一个单词，而是一个列表或者其他类型，文档里就没有讲。这里尝试了一下：

    curl -XPOST 'http://192.168.1.107:9200/cars/transactions/_bulk?pretty' -d'
    { "index": {}}
    { "price" : 10000, "color" : ["red", "green"], "make" : "honda", "sold" : "2014-10-28" }
    { "index": {}}
    { "price" : 20000, "color" : "red golden", "make" : "honda", "sold" : "2014-11-05" }
    { "index": {}}
    { "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
    { "index": {}}
    { "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
    { "index": {}}
    { "price" : 12000, "color" : ["green", "pink"], "make" : "toyota", "sold" : "2014-08-19" }
    { "index": {}}
    { "price" : 20000, "color" : "red yellow", "make" : "honda", "sold" : "2014-11-05" }
    { "index": {}}
    { "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
    { "index": {}}
    { "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }'
    
这里color有单词，字符串和列表，然后运行一下aggs

    curl -XGET 'http://192.168.1.107:9200/cars/transactions/_search?pretty' -d'
    {
        "size" : 0,
        "aggs" : { 
            "popular_colors" : { 
                "terms" : { 
                  "field" : "color"
                }
            }
        }
    }'


结果如下：

    {
      "took" : 16,
      "timed_out" : false,
      "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
      },
      "hits" : {
        "total" : 21,
        "max_score" : 0.0,
        "hits" : [ ]
      },
      "aggregations" : {
        "popular_colors" : {
          "doc_count_error_upper_bound" : 0,
          "sum_other_doc_count" : 0,
          "buckets" : [ {
            "key" : "red",
            "doc_count" : 12
          }, {
            "key" : "green",
            "doc_count" : 8
          }, {
            "key" : "blue",
            "doc_count" : 3
          }, {
            "key" : "pink",
            "doc_count" : 2
          }, {
            "key" : "golden",
            "doc_count" : 1
          }, {
            "key" : "yellow",
            "doc_count" : 1
          } ]
        }
      }
    }

实际上位于列表和字符串里的golden, 位于列表中的pink和green都被正确的检索到了。看来elasticsearch的aggs功能相当智能。
