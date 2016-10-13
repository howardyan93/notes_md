hadoop进坑记
========================

最近安装了hadoop，发现最初步的配置都遇到坑。。。

在.bashrc里设置

    export JAVA_HOME=/usr/lib/jvm/default-java

然后运行hadoop的命令

    start-dfs.sh

直接报错

    localhost: Error: JAVA_HOME is not set and could not be found.

上stackoverflow上查了一下，试了集中方法，都不行。

最后实在没办法了，直接在hadoop-config.sh里添加

    export JAVA_HOME=/usr/lib/jvm/default-java

终于可以运行了。

感觉这玩意真是坑。。

在stackoverflow也没有什么决定性的解决办法。

hadoop版本是2.6.4 
