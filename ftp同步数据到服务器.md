ftp同步数据到服务器
======================
现在有同步数据到服务器的需求。按道理来讲，一般是通过sftp来做最为合适，比如:

    scp -r /local/path user@host_name:/remote/path

在这里添加好ssh免密码或者使用sshpass即可。

但是客户回答说只能用ftp, 不能使用sftp。因为ssh登录密码将来会收回去。。。

这下就比较麻烦了，在网上搜了一下，此类问题早就[有人](https://segmentfault.com/a/1190000000777713)解决过了:

    #!/bin/bash 
    updir=/root/tmp
    todir=tmp
    ip=127.0.0.1
    user=username
    password=passwd
    sss=`find $updir -type d -printf $todir/'%P\n'| awk '{if ($0 == "")next;print "mkdir " $0}'` 
    aaa=`find $updir -type f -printf 'put %p %P \n'` 
    ftp -nv $ip <<EOF 
    user $user $password
    type binary 
    prompt 
    $sss 
    cd $todir 
    $aaa 
    quit 
    EOF

解决方法就是用find找出所有的目录，然后在ftp命令行里执行对应的创建。完成后继续用find命令找到文件放到远端创建好的文件夹里。

这个方法我用了一下，的确可用。但是我仍感觉命令稍显繁琐，而且每次执行都会覆盖原有的文件，或许有命令能检查重复文件来跳过，但是研究起来比较花时间。直到我发现机器上已安装了lftp工具，在ftp SHELL里直接一行搞定：

    lftp $username:$password@$host
    #此时进入ftp shell
    mirror -R /local/path /remote/path

就能自动同步本地文件到远程主机。

如果要编写脚本来自动执行，可以照着上面的套路这么写：

    lftp <<EOF
    open $username:$password@$host
    mirror -R /local/path /remote/path
    quit
    EOF

其实lftp本身有个-c的选项来读入命令：

    lftp -c "open $username:$password@$host && mirror -R /local/path /remote/path"

结束后会自动退出。

另外也可以通过管道来执行：

    echo "open $username:$password@$host && mirror -R /local/path /remote/path" | lftp

lftp使用上远比ftp命令好用，很多linux发行版默认安装的，除了debian。。。
