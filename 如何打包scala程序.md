如何打包scala程序
========================

最近开始学习scala，然后写了几个练手的程序，在scala下也能跑的起来。

但是在某些没有安装scala的主机上，又遇到不愿意安装scala的，就只能用java跑了。

这里就要用到scala打包程序。我也是参考了这个[链接](https://dzone.com/articles/wordcount-with-storm-and-scala).

以IDEA为例，首先在project文件夹下新建一个plugins.sbt文件。然后添加一下内容：

    addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.10")

然后就是运行一下命令：

    sbt clean compile assembly

如果没有设置生成assembly的包名的话，此时就会在target里生成一个名为xxx-assembly-0.1.jar的jar包，这个jar就是包含了内置scala以及其他依赖的宽包。

如果直接运行：

    java -jar xxx-assembly-0.1.jar

会直接报错：

    xxx-assembly-0.1.jar中没有主清单属性

因为在jar包的MANIFEST.MF里，并没有指定Main的。如果非要运行，可以编辑MANIFEST.MF,手动添加程序入口：

    Main-Class:  xxx.yyy.Main

如果偷懒，可以classpath的方式来跑：

    java -cp xxx-assembly-0.1.jar xxx.yyy.Main
    

