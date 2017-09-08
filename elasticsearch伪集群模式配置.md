elasticsearch伪集群模式配置
===================================
原来在服务器上玩过的是es集群。

最近感觉在测试服务器上单机es超级浪费，因为服务器内存128G，而es推荐的heap是不超过32G, 而这个测试服务器除了es也不干别的事情，于是想玩一下es的伪集群。

决定配置一个master节点，两个data节点。首先修改config文件夹下的jvm.options:

    -Xms30g
    -Xmx30g

master的配置直接网上找了一个来改：

    cluster.name: leopard
    node.name: master-1
    node.master: true
    node.data: false

    http.port: 9200
    transport.tcp.port: 9300
    discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300"]
    discovery.zen.minimum_master_nodes: 1

配置完后直接启动：

    ./bin/elasticsearch

跳了一个错：

    [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

这个问题以前在centos6这样的老版本的机器上遇到了，当时还费了不少力气改了很多参数。但是对于debian8, 参照es的github的[issue92](https://github.com/spujadas/elk-docker/issues/92)，只需要修改/etc/sysctl.conf, 添加

    vm.max_map_count=655360

即可。同时以root用户运行：

    sysctl -w vm.max_map_count=655360

让修改及时生效。

使用es启动命令，就可以看到es的master的主节点已经成功运行。

但是伪集群模式要让es读入不同的配置文件，这里参考[官网setting](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)

    ./bin/elasticsearch -Epath.conf=/path/to/my/config/
    
把config复制一份为config_master1。运行的时候加上选项：

    ./bin/elasticsearch -E path.conf=config_master1

这个时候再配置数据节点config_node1：

    cluster.name: leopard
    node.name: node-1
    node.master: false
    node.data: true
    node.max_local_storage_nodes: 3

    http.port: 9201
    transport.tcp.port: 9301

    discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300"]
    discovery.zen.minimum_master_nodes: 1
   
这里要注意一下几点：

    1. 因为偷懒，没有设置保存的data和log路径，这时候会显示默认的/data文件夹下文件被锁。所以要设置node.max_local_storage_nodes。如果是单独配置的，则不需要。
    2. http.port和transport.tcp.port要避免跟master使用的端口冲突。
    3. 由于transport.tcp.port改了之后默认用9301去连接Master，所以要强制指定discovery.zen.ping.unicast.hosts端口。

然后运行：

    ./bin/elasticsearch -E path.conf=config_node1

这个时候访问[web接口](http://127.0.0.1:9200/_cluster/health), 就能看到已经有一个master节点和一个node节点在线了。

如果是集群想要充分挖掘机器性能，也可以如法炮制。如果恰好shard和replica都分配在这台机器的master和node上，遇到机器挂了，会有数据丢失的风险，所以还需要设置好shard和replica的分配。


    

    
