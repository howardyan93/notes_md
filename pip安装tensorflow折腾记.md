pip安装tensorflow折腾记.md
========================

最近tensorflow风头一时无两。当初用pip安装的版本为0.8.0已经太旧了。现在吧python的环境都用virtualenv搞成虚拟环境了，于是决定重新安装：

    pip install tensorflow

结果报错：

    Downloading/unpacking tensorflow
        Could not find any downloads that satisfy the requirement tensorflow
    Cleaning up...
    No distributions at all found for tensorflow
    Storing debug log for failure in /xxx/.pip/pip.log

检查log发现，错误内容不是网络问题，而是找不到对应的版本：

    Could not find any downloads that satisfy the requirement tensorflow

目前的pip版本是1.5.6：

    pip -V

首先需要先更新pip：

    pip install --upgrade pip

完成后检查pip版本为9.0.1， 然后顺利安装tensorflow。

ps: 因为涉及到编译，必须先有python-dev库

    

系统的安全日志路径是：/var/log/secure

想了想还是修改ssh端口为妙。

ssh端口配置文件位于/etc/ssh/sshd_config

    # What ports, IPs and protocols we listen for
    Port 22
    # Use these options to restrict which interfaces/protocols sshd will bind to

修改端口号，然后运行service sshd restart即可。
