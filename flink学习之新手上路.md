flink学习之新手上路
========================

随着大数据的发展，flink也变得越来越火。于是我决定开始学习flink。flink本身对java, scala和python提供了支持。作为一个Pythonista，按道理我应该使用python，但是官方代码里面，涉及connector等部分都是基于java开发的，只有java和scala的例子，所以我决定还是用scala来学习。

### flink的前置条件
- linux系统。我的目前是debian10
- jdk8, 这里可以参考我以前写的如何安装JDK8的[文章](https://github.com/howardyan93/notes_md/blob/master/debian8%E6%97%A0%E7%97%9B%E5%AE%89%E8%A3%85JDK8.md)
- scala 2.11(主要是为了和flink版本对应)

### flink单机集群的搭建

一开始我从官网下载的是[flink_1.9.0](https://archive.apache.org/dist/flink/flink-1.9.0/flink-1.9.0-bin-scala_2.11.tgz), 但是隔了几天上去一看，都升级到[flink_1.9.1](https://www.apache.org/dyn/closer.lua/flink/flink-1.9.1/flink-1.9.1-bin-scala_2.11.tgz)。因为机器上的scala版本是2.11，我还是选择了1.9.0, scala2.11的版本

下载解压后，修改配置文件，因为是单击模式，这里只需要修改一点内容：

    rest.port: 8090
    rest.address: 0.0.0.0
    rest.bind-port: 8090-8100
    rest.bind-address: 0.0.0.0

有人会问为啥resp.port不是默认的8081, 因为这个端口我用来搞polipo代理了。。。

修改完成后进入bin目录，然后运行命令：

    ./start-cluster.sh

如果没有报错，用浏览器打开(集群主页)[http://localhost:8090], 就能看到flink的webui和一些集群基本信息。功能上有点类似与storm的web，展示了有多少可用资源，当前运行的任务以及可视化，在web上停止任务等。

### 配置环境变量

为了方便使用，需要把export flink的bin路径, 编辑～/.bashrc：

    export PATH=$PATH:/your_path/flink-1.9.0/bin

然后运行：

    source ~/.bashrc

接下来就可以开始正式进入flink学习之旅了。
