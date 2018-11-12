elasticsearch集群之间如何倒数据.md
==============================
elasticsearch集群之间倒数据真是一个头疼的问题。比如说自己在某个测试机器上配置了很多kibana的图形化展示，不想自己在新集群上再搞一遍，就需要在两者之间进行迁移。

有人会说，你先创建一个repo，然后snapshot, restore啊。

一开始就是这么搞的，但是遇到各种奇怪的问题：

    from elasticsearch import Elasticsearch
    es = Elasticsearch(hosts='localhost')
    snapshot_body = {"type":"fs", "setting":{"location":"/opt/elasticsearch/es_snapshot"}}
    es.snapshot.create_repository(repository="backup", body=snapshot_body)
    index_body = {"indices": ".kibana"}
    es.snapshot.create(repository="backup", snapshot="bk_2018_04_20", body=index_body)

这里因为用的是本地磁盘来保存snapshot。如果在elasticsearch.yml里repo.path没有设置路径为/opt/elasticsearch/es_snapshot的话，会报错：

    TransportError: TransportError(500, u'repository_exception', u'[backup] missing location')

保存完毕后，将backup文件夹拷贝到新的集群下的某个数据节点，然后配置同样的repo.path，运行：

    es.snapshot.restore(repository="backup", snapshot="bk_2018_04_20", body=index_body)

这个时候会报repository不存在的错误，所以还得先创建repository, 然后再把数据拷贝到对应文件夹下。这个时候再运行restore命令后，显示成功。这里可以通过命令来检查下情况：

    es.snapshot.status(repository='backup', snapshot="bk_2018_04_20")

但是实际发现这样倒数据会出现问题，打开kibana会显示找不到对应的数据。怀疑是数据没倒完全，因此删除只能删除repo重来：

    es.snapshot.delete_repository(repository='backup')
    es.snapshot.create_repository(repository="backup", body=snapshot_body)

这下直接坏菜了，直接报RepositoryMissingException, 显示子节点上没办法创建也找不到repo。我估计是用了本地硬盘的原因，而且其他子节点上没有配置repo.path。如果要搞定，还需要配完配置rollover，感觉非常蛋疼。这条路或许有巧妙的解决方案，但是官网上资料很少，所以只能放弃。

在stackoverflow上搜索了一下，有人用

    elasticdump --input=http://source_host:9200/my_index --output=http://dest_host:9200/my_index --type=data

但是隔离集群安装npm的东西麻烦，放弃。

最后发现elasticsearch在新版了提供了一个reindex的功能，专门提供两个集群之间迁移数据：

    es_src = Elasticsearch(hosts='host_src')
    es_dst = Elasticsearch(hosts='host_dst')
    index='.kibana'
    es_dst.indices.delete(index=index)
    elasticsearch.helpers.reindex(client=esSrc, source_index=index, target_index=index, target_client=esTar)

这样在打开kibana，使用上没有问题。

但是reindex有时候会自作聪明的修改index的mapping配置，尤其是对自己定义的index。所以建议自己定义的index, 可以在目标集群上建立好mapping后再进行迁移。

