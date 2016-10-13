ElasticSearch访问控 制的配置
========================

搭建了一个小的ElasticSearch集群来提供快速的索引服务。在调试完后，准备进入正式环境。因为ElasticSearch在集群中暴露了9200和9300端口，有心人很容易通过9200端口把你辛辛苦苦丢进去的索引数据删除掉。因为需要对访问进行控制。

官网推荐的方法就是使用它家的sheild工具，免费试用30天，之后开始收费。很明显这种小应用是掏不起这个价钱了，只能考虑通过iptables来进行控制。

stackoverflow上有现成的解答方法，

    iptables -I INPUT 1 -p tcp --dport 9200:9400 -s IP_ADRRESS_1,IP_ADRRESS_2,IP_ADRRESS_3 -j ACCEPT
    iptables -I INPUT 4 -p tcp --dport 9200:9400 -j REJECT
    sudo sh -c "iptables-save > /etc/iptables.rules"

集群的每台机器同时有一个内网地址和外网地址，内网地址的端口地址提供给elasticsearch的节点进行通信，外网地址直接封掉对应的端口。

但是实际操作中一加上配置集群立刻就找不到节点了，自己也是刚开始玩iptables，不明所以。后来试了很多方法，才找到一个配置：

    iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -A INPUT -p icmp -j ACCEPT

    iptables -A INPUT  -i lo -j ACCEPT
    iptables -A OUTPUT -o lo -j ACCEPT

    iptables -A INPUT  -i eth0 -j ACCEPT
    iptables -A OUTPUT -o eth0 -j ACCEPT
    iptables -A OUTPUT -o eht1 -j ACCEPT

    iptables -A INPUT -p tcp -m tcp --dport 9200:9400 -s 192.168.0.1/18 -j ACCEPT
    iptables -A INPUT -p tcp --dport 9200 -j REJECT
    iptables -A INPUT -p tcp --dport 9300 -j REJECT

    
