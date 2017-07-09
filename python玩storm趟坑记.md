python玩storm趟坑记
========================

准备开始学习storm。storm是最有名的实时流处理，spark虽然也有spark streaming，出来得比较晚，要趟的坑比storm要多，首选还是storm。

storm是核心是Clojure编写的，号称可以支持任何一种语言来编写拓扑。

由于只会玩python，所以还是得从python中选择对应的库文件。

Python目前有两个库，一个是pyleus，一个是streamparse。前者在github上已经有两年都不更新了，只支持到storm 0.9。后者一直在更新，所以对于最新的strom 1.1.0, 没有多的选项了。

storm本地伪集群我采用了[官网配置](http://storm.apache.org/releases/current/Setting-up-a-Storm-cluster.html)

    storm.zookeeper.servers:
         - "127.0.0.1"
    
    nimbus.seeds: ["127.0.0.1"]
    ui.port: 9090
    
    nimbus.host: "localhost"
    supervisor.slots.ports:
        - 6700
        - 6701
        - 6702
        - 6703

由于ui默认的8080端口被我用来搞polipo了，这里设置网页端口为9090.

然后按如下顺序启动storm即可：

    cd /xxx/yyy/apache-storm-1.1.0/bin
    ./storm nimbus &
    ./storm supervisor &
    ./storm ui &
    
打开localhost:9090，如果没啥问题，就能看到storm的管理页面。

然后开始安装streamparse，因为我使用了虚拟环境，所以无需sudo:

    pip install streamparse

然后开始跑demo：

    sparse quickstart wordcount
    cd wordcount
    sparse run

结果直接跳错，告诉我缺少lein，搞得我一脸懵逼。google了一下，才知道这是Clojure的包管理工具。于是直接去[lein官网](https://leiningen.org/#install)

lein的安装有两种方式，一种是用脚本下载安装，一种是要加PPA。原本lein也提供apt直接的安装了，结果各种历史原因，所以呵呵了。。。

作为懒人，首选脚本下载。结果速度奇慢无比。。。看着安装进度需要一天，我果断放弃。。。

只能选择PPA添加，apt安装了：

    sudo add-apt-repository ppa:mikegedelman/leiningen-git-stable
    sudo apt-get update
    sudo apt-get install lein

完成后继续跑sparse run命令。。结果还是不行，去stackoverflow翻了一通后发现，需要配置config.json。但是streamparse的demo里没提，说创建玩项目直接就能跑，我顿时感觉有点坑啊。。。

配置config.json如下:

    {
        "serializer": "json",
        "topology_specs": "topologies/",
        "virtualenv_specs": "virtualenvs/",
        "envs": {
            "prod": {
                "user": "user_xxx",
                "ssh_password": "password",
                "nimbus": "localhost",
                "workers": ["localhost"],
                "log": {
                    "path": "",
                    "max_bytes": 1000000,
                    "backup_count": 10,
                    "level": "info"
                },
                "virtualenv_root": "/path_xx/virtualenvs"
            }
        }
    }
    
然后试着运行:

    sparse run

然后立刻被打脸，说ssh user_xxx@localhost要输入密码，如果sshd_config如果没配置，即便输入正确的密码也会失败。这里可以参考我之前写的[如何ssh本地主机](https://github.com/howardyan93/notes_md/blob/master/%E5%A6%82%E4%BD%95ssh%E6%9C%AC%E5%9C%B0%E4%B8%BB%E6%9C%BA.md)

配置完免密码登录后，连密码一栏都不用搞了，再次运行。

机器会花一定时间来编译JAR文件，然后就能看到实时流的输出了。

但是这只是试运行，如果要发布拓扑到storm集群上，则要运行：

    sparse submit

结果又跳了一个错，说pip版本太低。。。

streamparse会在节点上构建python的虚拟环境, 然后在节点上安装好所有需要的python库。看脚本执行的顺序，会在生成虚拟环境后自动升级pip。但是不知道为何没有执行成功。所以我只能手动去对应的路径里升级pip:

    cd /path_xx/virtualenvs/wordcount/bin
    source activate
    pip install --upgrade pip
    deactivate

最后再次运行：

    sparse submit

没有报错就表示已经提交拓扑到storm上了，打开ui地址，可以看到拓扑一栏里已经显示有wordcount的拓扑在运行。
