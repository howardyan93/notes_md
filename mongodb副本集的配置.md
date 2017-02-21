mongodb副本集的配置
===================================
最近需要用到mongodb的副本集来进行数据的冗余备份。以前参照《mongodb权威指南》来玩过test replicaSet。

    replicaSet = new ReplSetTest({"nodes": 3})
    replicaSet.startSet()
    replicaSet.initiate()

导致印象当中，以为配置副本集就是在原来的mongo之外起一个，自动连主mongo。
带着这种认知，直接被带到沟里去了。怎么配的不对，最后才发现，这玩意儿跟elasticsearch的组织方式是类似的。
参照网上搜到的资料，配置如下：

    mongod --dbpath /path/mongo1 --logpath /path/mongo1/log.log --replSet shard1 --port 10001 --bind_ip 127.0.0.1 
    mongod --dbpath /path/mongo2 --logpath /path/mongo2/log.log --replSet shard1 --port 10002 --bind_ip 127.0.0.1

注意两个mongod存放数据和日志的路径，以及端口是不同的。
用mongo连接第一个mongod:
   
    mongo --port 10001

然后配置primary和secondary：

    cfg = {_id:"shard1", members:[{_id:0,host:'127.0.0.1:10001'}, {_id:1, host:'127.0.0.1:10002'}]}
    rs.initiate(cfg)
    rs.status()

这个时候基本上就可以了，其间遇到一些问题，原因是第二个mongod没有起得来。
如果再添加一个副本集, 除了按照上述方法之外，由于已经配置了rs，因此可以使用reconfig函数来更新副本集的配置：

    mongod --dbpath /path/mongo3 --logpath /path/mongo3/log.log --replSet shard1 --port 10003 --bind_ip 127.0.0.1
    cfg = {_id:"shard1", members:[{_id:0,host:'127.0.0.1:10001'}, {_id:1, host:'127.0.0.1:10002'}, {_id:2, host:'127.0.0.1:10003'}]}
    rs.reconfig(cfg)

假设我们在primary里插入一条记录：

    use my_test
    db.test1.insert({'greeting':'my lord'})

然后从secondary里查看：

    use my_test
    db.test1.find()

直接报错：

    error: { "$err" : "not master and slaveOk=false", "code" : 13435 }

原因在于mongo副本集实现了读写分离，默认的副本无法读写，需要在secondary里运行：

    rs.slaveOk()

这个时候就可以在副本中查询了。
当然，副本是无法进行写的。
    
    
    
    
    
    

