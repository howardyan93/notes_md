pymongo高级查询
===================================
最近玩了一下mongdb，用pymongo连接mongodb的某些查询跟直接在mongo_cli有些不同。记录如下:

    #模糊查询
    query = db.collection.find({"title": {"$regex": "xxx"}})
    #多个模糊查询
    query = db.collection.find({"title": {"$regex": "xxx|yyy"}})
    #过滤字段
    query = db.collection.find({}, {"_id":0, "key": 1})
    #时间范围查询
    query = db.collection.find({"post_date": {"$gte": "2016-01-01"}})
    query = db.collection.find({"post_date": {"$gte": "2016-01-01", "$lt": "2016-07-08"}})
    
    
    
    
    

