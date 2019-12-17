nim的IDE指南
===================================
### IDE工具的安装

虽然nim语言的语法简介明快，各种特性也很强大。但是如果没有一个强大的IDE工具，估计也会累得要死。比如笔者一开始就上手的时候就傻乎乎的在vim里面敲代码。问题是，nim语言不像其他语言是基于类开发的，很多方法没有绑定到对象上，要使用对应的方法，要先import对应的lib，对lib不熟悉，也没有自动补全的情况下，只能靠文档去看。就算是inim，目前也没有自动补全和自动提示的功能。

所以官方在项目wiki里就提供了[IDE工具的指引](https://github.com/nim-lang/Nim/wiki/Editor-Support)。

官方原来首推的工具是[Aporia](https://github.com/nim-lang/Aporia/)。但是这个项目在2018年就开始逐渐被遗弃，而且也不支持windows，后面官方强烈推荐全部转到了[vscode](https://code.visualstudio.com/)上。

首先安装vscode，笔者由于之前只用pycharm和idea，也是第一次用vscode。。。

安装完后需要安装对应的插件。打开vscode。按Ctrl+P，在命令框里输入:

    ext install kosz78.nim

等待安装完成就可以，这里也可以点击左侧菜单栏的extensions按钮，查看安装情况。

官方也推荐了其他的辅助工具:

    ext install oderwat.indent-rainbow
    ext install mattn.Runner
    ext install webfreak.debug
    ext install vscodevim.vim

**oderwat.indent-rainbow**是辅助显示缩进的工具，作为唯二的缩进语言，这个功能很好很强大; **mattn.Runner**用于执行代码里的脚本片段; **webfreak.debug**用来debug代码; **vscodevim.vim**是笔者自己的喜好。

安装完成后重启vscode。

### nim项目的创建

虽然在VSCODE里安装了nim插件，但是要像模像样的创建一个nim项目，笔者也没找到方法。最后只能借助nimble来实现。有人会问：什么？nimble不就是一个包管理工具么？的确，nimble除了支持安装卸载包，也支持创建并发布包。

首先创建一个空文件夹，比如说nimLearn，在命令行里运行：

    cd nimLearn
    nimble init

然后会有三个选项：library, binary, hybrid。这里按tab键选择binary。然后输入版本号，package描述，开放协议，后端语言和nim语言半本。我这里一路按Enter。

整个项目的结构如下：

    .
    ├── nimLearn.nimble
    └── src
        └── nimLearn.nim

项目会自动生成一个后缀为nimble的文件，一个src文件夹，里面包含了一个同名的nim文件。

nimLearn.nim自动生成的内容很简单，就是一个hello world：

    # This is just an example to get you started. A typical binary package
    # uses this file as the main entry point of the application.

    when isMainModule:
      echo("Hello, World!")


然后我们看一下nimLearn.nimble文件：

    # Package

    version       = "0.1.0"
    author        = "howardyan93"
    description   = "A new awesome nimble package"
    license       = "MIT"
    srcDir        = "src"
    bin           = @["nimLearn"]



    # Dependencies

    requires "nim >= 1.0.2"

其中的信息大部分信息就是我们init项目时候填入的信息。这里要注意bin的配置，里面指定了编译生成binary的名称。nimble在build的时候会根据srcDir去寻找同名的代码文件，然后生成对应的binary。

这个时候运行：

    nimble build

文件夹下就会生成一个nimLearn的可执行文件。

然后运行这个可执行文件：

    nimble run nimLearn

当然也可以直接运行:

    ./nimLearn

但是会少显示一些额外的信息。

创建完项目后，直接用vscode打开文件夹，就可以对代码进行编辑。这个时候自动提示和代码高亮等等功能都开启了，写起来会舒服很多。

实际使用nim插件的过程中，还是会发现有很多不尽如人意的地方。比如要查看某个函数的定义。直接点击Go to Definition有时候没啥响应。只有先点击Find All References，再点击Go to Definition才能跳转到函数定义。nim默认就引入的包，有时候也没有高亮的效果。只能指望插件作者以后改善了。

### DEBUG代码的方法

官网文章里介绍了DEBUG的[三种方法](https://nim-lang.org/blog/2017/10/02/documenting-profiling-and-debugging-nim-code.html):
- 用echo打印出来
- 用writeStackTrace
- 用GDB/LLDB

前面两种方法无需多言。第三种方法如果在命令行里搞，除非是高手，否则也会觉得蛋疼无比(反正笔者已经习惯了pyCharm和Idea里直接debug的方便了。。)。论坛里也有人请教nim插件作者是否支持debug功能，回答是肯定的。

我们给代码多添加点内容：

    import strutils
    import strformat

    when isMainModule:
        let myName = " howardyan93 "
        let pureName = myName.strip()
        var counter = 0
        while counter < 100:
            inc(counter)
            echo $counter
        echo(fmt"Hello, {pureName}!")

编译时添加额外的选项:

    nimble build --debuginfo --linedir:on

当然也可以用：

    nimble build --debugger:native

两者功能都一样，区别在与后者是新最新版nim才有的功能。

在vscode里点击左边的debug按钮，这个时候显示没有配置文件，选择add configuration，默认选择GDB。这个时候会在.vscode文件夹生成一个launch.json的文件。

打开launch.json文件。内容如下：

    {
        // Use IntelliSense to learn about possible attributes.
        // Hover to view descriptions of existing attributes.
        // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Debug",
                "type": "gdb",
                "request": "launch",
                "target": "./bin/executable",
                "cwd": "${workspaceRoot}",
                "valuesFormatting": "parseText"
            }
        ]
    }

这里要把target修改为刚才生成了nimLearn可执行文件：

    "target": "./nimLearn",

在代码里添加断点，然后点击Start Debugging。

这个时候就可以对代码进行debug了，debug操作跟debug其他语言差别不大。困难的是由于后端都是c，所以茫茫多的各种地址，看得头疼。我觉得我还是用前面两种方法比较后。gdb什么的，臣妾做不到啊。

### 后记

vscode的特色在于通过各种tasks来实现操作的自动化。

在终端菜单里点击Run Build Task, 这个时候会显示没有tasks文件，是否要创建一个，选择others，一路确定，就会在.vscode里生成一个tasks.json:

    {
        // See https://go.microsoft.com/fwlink/?LinkId=733558
        // for the documentation about the tasks.json format
        "version": "2.0.0",
        "tasks": [
            {
                "label": "echo",
                "type": "shell",
                "command": "echo Hello"
            }
        ]
    }

这个时候，我们可以修改为一个编译debug文件的task:

    {
        // See https://go.microsoft.com/fwlink/?LinkId=733558
        // for the documentation about the tasks.json format
        "version": "2.0.0",
        "tasks": [
            {
                "label": "build debug nim c",
                "type": "shell",
                "command": "nimble",
                "args": [
                    "build",
                    "--debugger:native",
                ],
                "group": {
                    "kind": "build",
                    "isDefault": true
                },
                "presentation": {
                    "reveal": "silent"
                },
                "problemMatcher": "$msCompile"
            }
        ]
    }

保存后删除原来的nimLearn可执行文件，然后在终端菜单里点击Run Task。之后会有一个菜单让你选择要运行的命令。由于我们只配置了一个task，所有会显示label为build debug nim c的选项，直接点击，就可以看到新生成的可执行文件。

结合vscode的task，很多操作就无需在命令行里进行了。但是nimble本身也支持task，同样的功能也能通过编辑nimLearn.nimble来实现:

    task GenerateDebugCode, "Compile debug file":
        exec "nimble build --debugger:native"

在命令行里运行：

    nimble tasks

就会显示出自定义的task以及描述。(另外直接运行nimble --help, task也会显示在帮助文件里。)

然后运行:

    nimble GenerateDebugCode

就能生成同样的debug可执行文件。

虽然vscode的task很好很强大。但是笔者更喜欢使用nimble的task，因为task的语法就是nim语法(这让我想到了kotlin和gradle)，写起来也很简单，配和一些语句可以实现很多666的玩法。当然两者搭配起来用也没啥问题。
