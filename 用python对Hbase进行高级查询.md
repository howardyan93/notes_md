用python对Hbase进行高级查询
========================

python访问Hbase虽然有很多库，但是目前最好用的还是happybase。happybase文档上手容易，但是很多高级查询没有一个详尽的文档。因此要玩转高级查询，还需要自己去翻阅[Hbase的thrift api文档](http://hbase.apache.org/book.html#thrift)

首先创建链接：

    import happybase
    conn = happybase.Connection('localhost')
    table = conn.table('table_xxx')

简单的scan查询：

    query = table.scan()
    result = next(query)

scan查询返回一个迭代器，理论上会返回一个表里面所有的结果。如果需要做限制，则需要加上limit参数：

    query = table.scan(limit=10)
    result = list(query)
    
如果要查找指定列簇某个列内容的匹配，则需要用到filter:

    query_str = "SingleColumnValueFilter ('a', 'aa', =, 'substring:test')"
    query = table.scan(filter=query_str, limit=10)

这里查询列簇a的列aa的内容是否包含'test'。

但是实际查看结果，发现某些不包含'aa'这个列的row也被返回了，这是因为api就是这么规定的：

    This filter takes a column family, a qualifier, a compare operator and a comparator. If the specified column is not found – all the columns of that row will be emitted. If the column is found and the comparison with the comparator returns true, all the columns of the row will be emitted. If the condition fails, the row will not be emitted.

所以在需要过滤掉这些不包含'aa'的内容时，需要添加过滤的参数：

    query_str = "SingleColumnValueFilter ('a', 'aa', =, 'substring:test', true, false)"
    
参数定义如下：

    SingleColumnValueFilter('<family>', '<qualifier>', <compare operator>, '<comparator>', <filterIfColumnMissing_boolean>, <latest_version_boolean>)

这个在Hbase的thrift api里面没有直接提及，但是在例子里有。感觉真是坑。。。

多个filter的逻辑组合：

    query_str = "SingleColumnValueFilter ('a', 'aa', =, 'substring:test', true，false) OR SingleColumnValueFilter ('a', 'aa', =, 'substring:check', true, false)"
    
这样能查询出a.aa这个列里面包含test或者check的row。

上面的逻辑可以通过一个正则来搞定：

    query_str = "SingleColumnValueFilter ('a', 'aa', =, 'regexstring:(test|check).com', true, false)"

filter匹配方式还有binary，binaryprefix。而filter还有RowFilter， ValueFilter等，这里就需要自己参考thrift文档来搞了，基本上和SingleColumnValueFilter大同小异。

