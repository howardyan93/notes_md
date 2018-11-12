solrcloud快速上手指南
===================================
以前都是使用elasticsearch来进行索引，但是最近某些情况不得不用的solr了。比如要使用这样的通配符检索信息: \*world来匹配helloworld。这个在目前的elasticsearch是没什么好的优化手段的，文档里也是建议要避免使用左通配这样的模式匹配。但是在solr里，solr搞了一种ReversedWildcardFilterFactory，将字段保存两份，其中一份倒序列排列来加速左通配。

由于solr单击和solrcloud使用还很不一样。实际需求是要用solrcloud来实现集群，因此直接从solrcloud直接上手。以下操作其实就是solr7.5的Tutorial的翻译：

####启动solrcloud集群

首先[下载](https://www-eu.apache.org/dist/lucene/solr/7.5.0/solr-7.5.0.tgz)sorl7.5的安装包，然后解压。

    unzip -q solr-7.5.0.zip

然后启动solr的cloud模式：

    cd solr-7.5.0/
    ./bin/solr start -e cloud

由于solr经过多年发展，算是做的比较人性化了，所以启动命令执行之后，会出现一步一步的提示，根据提示往下走就可以了：

    Welcome to the SolrCloud example!

    This interactive session will help you launch a SolrCloud cluster on your local workstation.
    To begin, how many Solr nodes would you like to run in your local cluster? (specify 1-4 nodes) [2]:

这里集群默认2是两个节点，直接按Enter。

    Ok, let's start up 2 Solr nodes for your example SolrCloud cluster.
    Please enter the port for node1 [8983]:

这里让选择第一个节点的端口，直接按Enter。

    Please enter the port for node2 [7574]:

第二个节点的端口，直接按Enter。然后要等一会，让两个节点启动起来。

    Now let's create a new collection for indexing documents in your 2-node cluster.
    Please provide a name for your new collection: [gettingstarted]

这里要输入新的Collection名，默认是gettingstarted, 输入techproducts，按Enter。

    How many shards would you like to split techproducts into? [2]

这里要求选择Shard的数量，直接Enter。

    How many replicas per shard would you like to create? [2]

这里要求选择Replica即备份书，直接Enter。

    Please choose a configuration for the techproducts collection, available options are:
    _default or sample_techproducts_configs [_default]

这里要求选择Collection的schema，\_default里面只是一个基本的骨架，为了配合后面的步骤，这里选择sample_techproducts_configs，里面已经配置好了很多字段。

到这一步solr cloud集群都启动起来了。然后命令行里会提示你去访问如下网页：

    SolrCloud example running, please visit: http://localhost:8983/solr

用浏览器访问，就能看到solr的web管理界面了。由于是cloud模式，所以没有core的菜单，取而代之的是Collections。

####导入样本数据

直接运行：

     bin/post -c techproducts example/exampledocs/*

就可以看到大量的xml数据导入到sorlcloud。初次之外，还可以直接导入json，csv等多种格式。


####查询及其它

由于查询涉及的东西比较大，这个放到以后研究。
