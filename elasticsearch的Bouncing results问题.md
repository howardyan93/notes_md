elasticsearch的Bouncing results问题
===================================
以前使用elasticsearch，排序上设定了以时间+score的排序方式:

    sort_setting = [
        {"pub_date": {"order": "desc"}},
        {'_score': {"order": "desc"}},
    ]

搜索结果也一直没有问题。但是最近一个项目，通过爬虫爬取后的数据不停的入ES。同样的搜索条件返回的不同的结果。

下面就是分析思路：

    1.排除es集群的问题，因为之前遇到过es设置不对，导致同一查询在不同节点返回的结果不一致的情况。把es的hosts设置为单一主机，结果仍然是会变化。
    2.限定查询的时间范围，如果日期的上线为今天，那么不停入库的数据肯定会导致结果的变化，限定为过去的时间段后，毫无改善。
    3.去掉pub_date排序字段，限定为按关联度排序，毫无改善。
    4.最后居然开始怀疑merge的问题，使用forcemerge，结果表示没啥用。

这个时候我才意识到了文档里提过的Bouncing Results问题。因为时间格式为%Y-%m-%d，那么同样时间的数据会有很多。es如果不做任何设置，将会按round-robined的方式从primary和replica里取了再排序，这样结果就不能保证每次都一样的。毕竟primary有的relica里不一定有，尤其是在不停往es里丢数据的情况。

最后解决方法也很简单，直接设置preference为primary即可：

    search_result = es.search(index=index_name,
                              body=search_body, preference="primary")

最大的担心就是这样会有性能问题，不过没时间来测。



