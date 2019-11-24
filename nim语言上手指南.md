nim语言上手指南
========================

最近跟人闲聊的时候了解到了一个非常独特的编程语言nim。语法风格跟python和pascal类似，但是执行效率却跟C/C++一个档次，而且跟Golang一样可以打包成平台独立的可执行文件。

作为一个Pythonista，我表示很有兴趣。

### nim的安装
首先去[官网](https://nim-lang.org/)看了一下安装过程。其实自己用的debian10上用apt也能够安装，但是版本不是最新的。这里我选择了直接下载压缩包，手动安装。

    wget https://nim-lang.org/download/nim-1.0.2-linux_x64.tar.xz)

下载完成后解压到某个目录，然后在.bashrc里添加环境变量:

    export PATH=$PATH:/data/tools/nim-1.0.2/bin


### 包的管理

nim的包管理软件叫[nimble](https://github.com/nim-lang/nimble)，已经随nim的安装包一起发布了。所以可以直接使用。具体操作可以参考github上的readme。

搜索，安装，卸载命令跟pip差不多，可以快速上手。



### 交互环境

以前初学python，全靠ipython这样的repl来学习基础的语法，能够快速看到结果并纠正错误。那么nim语言也有这样的类似的东西么？我用

    nimble search script

搜索了一下，还真有，而且叫inim(这是在致敬ipython么?)。安装也很简单：

    nimble install inim

一路点击yes。

按照[inim](https://github.com/AndreiRegiani/INim)里面所说的，安装完之后直接在命令行里输入inim就可以启动。但是我这边却失败了，而且全程安装没有要求sudo权限，我估计是安装在某个local的路径下了。于是我搜索了一下：

    find ~/ -name "inim"

发现安装在里用户目录的.nimble/bin/里，于是添加路径到环境变量:

    export PATH=$PATH:~/.nimble/bin/

搞定。


### 脚本执行

nim也可以实现脚本执行，但是需要安装包。搜索了一下，发现实现脚本执行的包还挺多。于是我按github的star选了一个最高的，nimr，安装也很容易：

    nimble install nimr

然后在自己编写的nim脚本第一行加上

    #!/usr/bin/env nimr


### nim的学习资源

在搜索nim学习系资源的时候，发现目前nim首推是官网的教程，其次有一个英文书籍<Nim in Action>, 中文有人翻译了[Core-Nim-programming](https://github.com/ScxMes/Core-Nim-programming/blob/master/目录.md)，感谢翻译大佬给我们铺平了道路。
    
