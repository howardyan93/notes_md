python原生访问hdfs文件系统
===================================
对于HIVE，生成orc或者parquet文件格式放在hdfs文件系统上，对外通过SQL语句，就能实现离线分析数据仓库的功能。

但是对于dask这样的python库，通过安装对应的hdfs库文件，也能做到跟HIVE一样的效果。无非就是不能支持SQL查询，取而代之采用dataframe的操作。

python访问hdfs库需要安装hdfs3库，而这个库又需要安装libhdfs3。

首先安装libhdfs3:

    sudo apt-get install libhdfs3 libhdfs3.dev

结果报了一个错：

    libhdfs3 : 依赖: libprotobuf8 但无法安装它
    E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系。

查看了一下，目前使用的libprotobuf版本都到9了。但是这个库依赖版本8。[github](https://github.com/Pivotal-Data-Attic/pivotalrd-libhdfs3)上看了一下, 似乎没有最新的libprotobuf9的代码，只能强行安装一波版本8。

直接去ubuntu下载deb包[libprotobuf8_2.5.0-9ubuntu1_amd64.deb](http://ubuntu.cs.utah.edu/ubuntu/pool/main/p/protobuf/libprotobuf8_2.5.0-9ubuntu1_amd64.deb)

然后dpkg安装：

    sudo dpkg -i libprotobuf8_2.5.0-9ubuntu1_amd64.deb

安装好后再通过安装libhdfs3。

最后安装python对应的lib:
    
    sudo pip install hdfs3

最后就可以按hdfs3的example来访问hdfs了。

不过看了一下，貌似dask虽然在例程中采用了hdfs这个库，但是也可以采用snakebite这个原生python库。比较少折腾。。

