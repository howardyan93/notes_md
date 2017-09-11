elasticsearch超省存储空间的配置
===================================
最近在研究elasticsearch 5.x版本的新特性的时候，偶尔翻到一篇es的测试文章，对于mapping的配置直接干掉了_all和_source选项。

对于_all选项的关闭我倒是可以理解，但是如果关掉_source选项的话，那返回数据就是一堆空的字典了。

所以看到这篇文章，我的第一感觉是: 卧草，还有这样的骚操作？！这是完全是把es当索引用了啊。

但是对于Hbase和mongodb存储的数据，这样搞也没什么坏处。通过索引拿到rowkey/_id, 然后直接从Hbase/mongodb里取数据。

这里我做了如下实验，首先配置mapping:

    mapping_body = {
        "article": {
            "_all": {"enabled": False},
            "_source": {"enabled": False},
            "properties": {
                # 各种字段。。。
            },
        },
    }

然后把mongo中的数据导入到es中。一共666380条数据，id为数据的唯一字段ac_no, 存储空间占了18M。

首先进行普通查询：

    search_result = es.search(index=index_name,
                              body=search_body, preference="primary")

由_source禁止掉了，所以返回的数据内容都是空，所以只能拿到id。通过id再在mongodb中进行$in查询:

    result = search_result[u'hits'][u'hits']
    id_list = [int(i["_id"]) for i in result]
    query = list(coll.find({"ac_no":{"$in":id_list}}, {"_id": 0}))

最后得到所有数据。要得到跟_source关闭前一样的结果，还需要做点处理，保证结果排序的一致，这里不再赘述。

如果_source开启，存储同样的数据需要28M，节省的空间还是比较显著的。

