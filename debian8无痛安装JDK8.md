debian8无痛安装JDK8
========================

elasticsearch更新到5.x版本后，都需要用JDK8。本来是件忒简单的事情，但是放在VPS上就有点懵了。apt-cache里搜不到。而工作用的电脑当时怎么装的也不记得的了。。。

按常理而言，可以直接下载jdk的安装包来搞定。但是完事儿后还要设置几个环境变量，比如这样：

    export JAVA_HOME=/home/jdk1.8.0_121
    export PATH=$JAVA_HOME/bin:$PATH
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    
时间长了鬼才记得住啊。还是得靠apt-get来搞比较轻松啊。VPS上新系统安装步骤如下：

1.先配置好/etc/apt/sources.list, 这里我用了网易的源：

    deb http://mirrors.163.com/debian/ jessie main non-free contrib
    deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
    deb http://mirrors.163.com/debian/ jessie-backports main non-free contrib
    deb-src http://mirrors.163.com/debian/ jessie main non-free contrib
    deb-src http://mirrors.163.com/debian/ jessie-updates main non-free contrib
    deb-src http://mirrors.163.com/debian/ jessie-backports main non-free contrib
    deb http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
    deb-src http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib

2.然后更新apt

    apt-get update
    apt-get upgrade

3.然后运行
    
    apt install -t jessie-backports  openjdk-8-jre-headless ca-certificates-java

4.最后安装jdk8

    apt-get install openjdk-8-jre

5.最后可以设置JAVA_HOME的环境变量，我偷懒，直接修改：

    ln -s /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java /etc/alternatives/java
    
有人可能会说，你这步骤比直接装要多啊。但是其实只需要执行3, 4, 5。新装vps的source list你总是要配置的。 
    
