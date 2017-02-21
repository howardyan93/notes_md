mongo-connector上手指南
===================================
从mongo倒数据到elasticsearch也是一件挺烦人的事情。虽然听说有同步工具，但是因为没什么时间，所以一直是写脚本从mongo到elasticsearch。

但是最近有点空了，开始研究如何偷懒。。。

mongo到elasticsearch最早的工具是river。但是后期由于不维护，被mongo-connector取代。

由于mongo这样的数据库没有触发器，因此无法像mysql那样直接玩。mongo-connector的作者采用了replicaSet，在副本集从primary同步数据的时候，通过oplog来知道有哪些新插入的数据，最后同步到elasticsearch。因此，同步有一定的延迟性。

当前的elasticsearch版本是5.1.1, mongo版本2.4.10

首先安装mongo-connector

    pip install mongo-connector

因为elasticsearch的缘故，还需要安装对应的doc manager：

    pip install 'mongo-connector[elastic5]'

完成后按照文档同步数据库, 这里mongo副本的配置参照之前的文章:

    mongo-connector -m localhost:10001 -t localhost:9200 -d elastic2_doc_manager

这里需要注意，由于版本的不同，使用的是elastic2_doc_manager。而不是elastic_doc_manager
    
连接es, 就能看到有新的索引创建好并同步了数据：

    from elasticsearch import Elasticsearch
    es = Elasticsearch()
    es.indices.get_alias()
    es.search(index='my_test')

mongo带权限的同步还没研究过。等有空再说。
