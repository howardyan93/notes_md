搭建本地debs源安装clickhouse
==================================
最近在学习clickhouse，在单机版上玩了一段时间之后，决定搭建clickhouse cluster。我买了4台intel的nuc，型号是6CAYH，每台机器8G内存，host配置如下(突然觉得自己无聊到蛋疼)：

    192.168.1.20    nuc6master1
    192.168.1.21    nuc6slave1
    192.168.1.22    nuc6slave2
    192.168.1.23    nuc6slave3

除了nu6master1配置了256G的SSD之外，其他的机器配置1T的5400转的笔记本硬盘。

然后按clickhouse的观望，配置apt源，然后下载：

    sudo apt-get install dirmngr
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4

    echo "deb http://repo.yandex.ru/clickhouse/deb/stable/ main/" | sudo tee /etc/apt/sources.list.d/clickhouse.list
    sudo apt-get update

    sudo apt-get install -y clickhouse-server clickhouse-client

但是遇到一个奇葩的问题，家里的联通网访问下载速度及其慢。每天机器安装玩估计都要6个小时！

当我用意外手机做热点的时候，发现下载速度就很快，几分钟搞定。然而4台机器，如果都要用手机搞，实在是太累了(你能想象用ssh进去然后配置wifi么？)。最好的办法就是下载deb包到本地，然后用apt安装。

首先检查deb的包依赖：

    apt depends clickhouse-server
    apt depends clickhouse-client

然后根据上面显示的包依赖关系，下载需要的所有的clickhouse deb包:

    apt download clickhouse-common-static clickhouse-compressor clickhouse-server clickhouse-client

然后我上传deb包到nuc6master1上。在root下执行：

    apt install ./clickhouse-compressor_1.1.54318_amd64.deb

然后apt显示要优先从官网下载。。。。继续几kb下载中。。。。

本来这种情况，直接删除/etc/apt/sources.list.d/clickhouse.list，然后apt update，重新再跑一次命令就可以了。但是我蛋疼的想法又来了，想想万一以后要升级什么的，难道还要重新搞一遍? 自然是自己搭一个本地deb源在将来比较省力。

论静态文件服务器，自然首选nginx。在nuc6master1上以root运行：

    apt install nginx-full

然后编辑/etc/nginx/sites-available/default，修改为:

    location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
                autoindex on;
                autoindex_localtime on;
	}

然后创建文件夹：

    mkdir -p /var/www/html/debs/clickhouse/

重启nginx：

    /etc/init.d/nginx restart

这个时候浏览器访问nuc6mater1的ip，显示nginx默认页面就说明修改成功。然后继续访问http://nuc6master1/debs/，就能看到文件层级目录。

然后把所有的clickhouse的deb包拷贝到/var/www/html/debs/clickhouse/下载。然后运行:

    cd /var/www/html/debs/
    dpkg-scanpackages clickhouse/ /dev/null | gzip> clickhouse/Packages.gz

dpkg-scanpackages后面为啥要接/dev/null，我也没功夫研究，直接照抄网上的。如果显示找不到dpkg-scanpackages，则需要先安装：

    apt install dpkg-dev

到目前位置私有deb原就搭建完成了，用浏览器访问对应的地址，如果显示正常就说明没啥问题。

最后配置apt的sourcelist。

    echo "deb [trusted=yes] http://nuc6master1/debs/ clickhouse/" > /etc/apt/sources.list.d/clickhouse_local.list

由于是本地源，没有制作Release文件，所以必须要加上[trusted=yes]， 否则会因为安全问题，apt禁止从这个源安装。我之前用debian9.2的时候，会报Warning，但是可以强制安装。debian10之后貌似安全性管理更加严格了。

最后就是安装clickhouse:

    apt update
    apt install clickhouse-client clickhouse-server

这里会显示输入密码，输入自己的密码或者直接Enter。。。

安装完成之后，运行客户端：

    clickhouse-client

这个时候会显示：

    Connecting to localhost:9000 as user default.
    Code: 210. DB::NetException: Connection refused (localhost:9000)

然后我尝试用密码登录：

    clickhouse-client --password xxxx

登录失败，然后我发现，无论是在安装时是否输入密码，都是不行的。需要先编辑/etc/clickhouse-server/users.xml， 在default层级下面添加密码：

    <password>xxxx</password>

然后重启clickhouse-server:

    /etc/init.d/clickhouse-server restart

然后就可以用密码登录了。

