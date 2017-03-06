mongo-connector带认证的同步
===================================
上次研究了mongodb不带帐号密码同步到es。但是通常情况下，为了安全，我们使用的mongodb都是带帐号密码的。

如果是只同步一个单击mongodb到es，其实设置很简单。

首先配置/etc/mongodb.conf文件

添加如下字段

    replSet=shard1

然后在终端里运行

    sudo service mongodb restart

完成后进入mongo-shell，创建用户名和密码：

    use admin
    db.addUser(username, password)

再修改一次/etc/mongodb.conf文件，设置为需要帐号密码登录模式:

    auth = true

这个时候，单机的副本集mongo就创建好了。然后按照之前的方法搞定mongo-connector, 当然这次要带上帐号和密码:

    mongo-connector -m localhost:27017 -t localhost:9200 -d elastic2_doc_manager --admin-username username --password password

如果是有多个mongo开启副本集模式，在配置mongo这一块稍微复杂点。这里先在/var/lib创建文件夹mongo_cluster. 

在mongo_cluster下建立文件夹mongo1和mongo2.

在mongo_cluster下建立一个集群的认证文件keyfile。里面填入集群认证密码。

接下来的一步很关键，需要将keyfile设置为只读模式，否则会报错：

    permissions on /var/lib/mongo_cluster/keyfile are too open

设置命令为：

    chmod 600 keyfile.

完成后执行：

    mongod --dbpath /var/lib/mongo_cluster/mongo1 --logpath /var/lib/mongo_cluster/mongo1/log.log --replSet shard1 --port 10001 --bind_ip 127.0.0.1 --oplogSize 64 --keyFile /var/lib/mongo_cluster/keyfile

然后进mongo-shell设置帐号和密码，方法跟单击副本集一直。

完成后再执行：

    mongod --dbpath /var/lib/mongo_cluster/mongo1 --logpath /var/lib/mongo_cluster/mongo1/log.log --replSet shard1 --port 10001 --bind_ip 127.0.0.1 --oplogSize 64 --keyFile /var/lib/mongo_cluster/keyfile --auth --fork
    mongod --dbpath /var/lib/mongo_cluster/mongo2 --logpath /var/lib/mongo_cluster/mongo2/log.log --replSet shard1 --port 10002 --bind_ip 127.0.0.1 --oplogSize 64 --keyFile /var/lib/mongo_cluster/keyfile --auth --fork

让mongo2跟mongo1同步即可。

这个时候再运行mongo-connector, 连接到任意一台mongo上。就能开始同步。
