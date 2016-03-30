mysql如何安全地删除用户
===================================
为了建立xmpp服务器，我先准备好mysql服务器。

创建用户很简单，但是因为创建的用户名不满意，决定删除重建。

删除用户的命令不常用，忘得差不多了，决定百毒一下。

头一条就是说直接去mysql.user下面删。虽然这个说法感觉很诡异(非常的不优雅啊有木有），但是一看回答日期是2015年，应该靠谱。

于是我按照说法直接输命令：

    delete from mysql.user where User=’user_name’;

然后

    flush privileges;

再次创建的时候，出麻烦了，直接提示：

    ERROR 1396 (HY000): Operation CREATE USER failed for ‘xxx”@’localhost’

明显是直接删除没有删干净的感觉。直接用英文去搜索，一个老外的回答是用drop user命令。

直接运行：

    drop user xxx@localhost

然后搞定。

百毒这玩意真心不靠谱。。网上都是些什么乱七八糟的答案。果然外事绝不问百度是至理名言。。。

