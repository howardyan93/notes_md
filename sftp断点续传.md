sftp断点续传
========================

最近一个同步数据的需求，需要把压缩包自动上传到VPS上。

基于安全性的考虑，不能使用FTP，就只能走SFTP协议。而且考虑到网络卡断的问题，还必须要有断点续传的功能。

用python操作sftp，自然首选是paramiko库。操作就跟之前操作FTP类似。但是断电续传什么的可就难住我了，因为一直在玩文件操作，也就是open, 然后read/write, 最后close。

SFTP断电续传什么的，臣妾做不到啊。

只能求助stackoverflow了，一个达人指点我说，可以用计算已上传的文件大小，然后在把缺少的补上。原理大家都懂，这个关键是这个达人告诉了我对应的SFTP的函数。

于是写出如下代码：

    # -*- coding:utf-8 -*-
    import config
    import paramiko
    import os
    privatekeyfile = os.path.expanduser('~/.ssh/id_rsa')
    mykey = paramiko.RSAKey.from_private_key_file(privatekeyfile)
    transport = paramiko.Transport((config.host, config.port))
    transport.connect(username=config.username, pkey=mykey)
    
    sftp = paramiko.SFTPClient.from_transport(transport)
    
    remot_dir = '/home/ddb/yanhao/sftp_test/'
    local_dir = '/data/home/yanhao/'
    filename = 'probe_api_server.tar'
    file_list = sftp.listdir(remot_dir)
    
    if filename in file_list:
        stat = sftp.stat(remot_dir + filename)
        f_local = open(local_dir + filename)
        f_local.seek(stat.st_size)
        f_remote = sftp.open(remot_dir + filename, "a")
        tmp_buffer = f_local.read(100000)
        while tmp_buffer:
            f_remote.write(tmp_buffer)
            tmp_buffer = f_local.read(100000)
        f_remote.close()
        f_local.close()
    else:
        f_local = open(local_dir + filename)
        f_remote = sftp.open(remot_dir + filename, "w")
        tmp_buffer = f_local.read(100000)
        while tmp_buffer:
            f_remote.write(tmp_buffer)
            tmp_buffer = f_local.read(100000)
        f_remote.close()
        f_local.close()
    
    sftp.close()
    transport.close()

我测试了一下，手动断网再链接，压缩文件正确上传玩了。不放心的同学还可以搞个MD5校验下。
