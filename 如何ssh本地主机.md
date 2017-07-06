如何ssh本地主机
========================

开始玩大数据的东西之后，发现很多东西都需要ssh登录本地机器。比如hadoop, 比如streamparse。在debian8默认的情况下

    ssh localhost

会提示输入密码，即便你输入了正确的密码，也会失败。

先去google了一把，第一个解决方案是修改/etc/ssh/sshd_config，先试着解决root的ssh问题：

    PermitRootLogin no

修改为

    PermitRootLogin yes

然后试着ssh登录root用户：

    ssh root@localhost

可以登录。但是问题是一直用root用户跑应用程序总是不安全。继续搜索，在sshd_config里添加：

    AllowUsers user_xx

重启ssh登录后登录:

    ssh user_xx@localhost

还是提示失败。查看了一下登录的日志记录：

    vim /var/log/auth.log

里面最新的日志里有一条

    Jul  3 15:30:12 debian sshd[15743]: User user_xx not allowed because shell bash does not exist

不知道当初创建任务时候指定了用什么shell。只能硬着头皮在root用户下修改user_xx的配置：

    usermod user_xx -s /bin/bash

然后继续登录，输入密码后登录成功。

但是输入密码登录也是麻烦事情。这里添加ssh key之后就可以免密码登录了：

    ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
    cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

感觉大数据之路上的坑真是多。。。

    
