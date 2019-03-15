如何用gradle创建scala项目
===================================

虽然scala的标配构建工具是sbt，但是随着kotlin的大火，越来越多的人也在使用gradle。所以就面临一个问题，用习惯gradle的人如何创建scala项目？

直接google了一番，很多文章说了半天，就是在讲怎么在IDEA里面配置，更勇的直接手动创建文件，感觉是在是太累了。

现在gradle的官网其实已经有如何创建scala项目的[指导](https://guides.gradle.org/building-scala-libraries/)了

首先在确保已经安装了gradle。然后创建项目文件夹

    mkdir ScalaExample

然后

    cd ScalaExample
    gradle init --type scala-library

这个时候，gradle就会自动创建好项目，按提示输入一些选项，我这里除了选择groovy作为gradle模版语言，其他都是一路Enter。

这个时候，build.gradle的内容如下：

    plugins {
        // Apply the scala plugin to add support for Scala
        id 'scala'
    }
    
    repositories {
        // Use jcenter for resolving your dependencies.
        // You can declare any Maven/Ivy/file repository here.
        jcenter()
    }
    
    dependencies {
        // Use Scala 2.12 in our library project
        implementation 'org.scala-lang:scala-library:2.12.7'
    
        // Use Scalatest for testing our library
        testImplementation 'junit:junit:4.12'
        testImplementation 'org.scalatest:scalatest_2.12:3.0.5'
    
        // Need scala-xml at test runtime
        testRuntimeOnly 'org.scala-lang.modules:scala-xml_2.12:1.1.1'
    }

由于我目前的scala版本是2.12.8，所以要修改一下对应的版本。

然后配置一下软件源，国内推荐是阿里，因此修改为：

    repositories {
        // Use jcenter for resolving your dependencies.
        // You can declare any Maven/Ivy/file repository here.
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
        mavenCentral()
        jcenter()
    }

这样基本就可以。其他更详细的配置就需要参考gradle的文档。
