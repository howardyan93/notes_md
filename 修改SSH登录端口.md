修改SSH登录端口
========================

这个站还才开始跑第一天，查看系统日志就发现有不少人在试我的ssh登录root密码。

系统的安全日志路径是：/var/log/secure

想了想还是修改ssh端口为妙。

ssh端口配置文件位于/etc/ssh/sshd_config

    # What ports, IPs and protocols we listen for
    Port 22
    # Use these options to restrict which interfaces/protocols sshd will bind to

修改端口号，然后运行service sshd restart即可。
