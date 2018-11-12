hadoop单机版的搭建
========================

从debian8升级到debian9后，一不小心把原来的hadoop环境给搞没了，只能重新搭建。

hadoop版本2.6.5

hdfs存储路径为/data/hdfs, 里面创建了对应的tmp和name文件夹。

首先配置hadoop-2.6.5/etc/hadoop/路径下的core-site.xml:

    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://localhost:9000</value>
        </property>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/data/hdfs/tmp</value>
        </property>
    </configuration>
    
再配置hdfs-site.xml:

    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>1</value>
        </property>
        <property>
            <name>dfs.name.dir</name>
            <value>/data/hdfs/name</value>
        </property>
        <property>
            <name>dfs.data.dir</name>
            <value>/data/hdfs/data</value>
        </property>
    </configuration>

最后配置mapred-site.xml：

    <configuration>
        <property>
            <name>mapred.job.tracker</name>
            <value>localhost:9001</value>
        </property>
    </configuration>

这样单机版的hadoop就基本配置好了。

然后格式化hdfs:

     bin/hdfs namenode -format

完成后就直接启动hadoop：

    sbin/start-all.sh

如果启动没有错误，就可以通过web ui来查看hadoop的运行情况:

    http://localhost:50070

运行命令，就能看到hdfs的实际使用情况：

    bin/hadoop fs -df /
