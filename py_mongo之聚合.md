pymongo之聚合
===================================
虽然elasticsearch的聚合功能很好很强大，但是对于一般的小项目，mongodb的聚合功能也是堪用的。

mongodb的聚合类似于管道操作，通过多个构件来组成一个管道：filter, project, group, sort, limit, skip。

sort, limit, skip在普通查询中也是经常使用的命令，这里主要介绍前面三个构建的使用。

比如要查找field1或field2正则匹配某个字符串，然后对field3, field3进行分组并统计出现的次数，可以按如下来搞：

    import pymongo
    
    db_cli = pymongo.MongoClient(host=mongo_host, port=mongo_port) 
    coll = db_cli["my_test"]["test_coll"]

    aggs = [
        {"$match": {"$or" : [{"field1": {"$regex": "regex_str"}}, {"field2": {"$regex": "regex_str"}}]}},
        {"$project": {"field3":1, "field4":1}},
        {"$group": {"_id": {"field3": "$field3", "field4":"$field4"}, "count": {"$sum": 1}}},
    ]
    
    result = coll.aggregate(pipeline=aggs) 
    
    
    

