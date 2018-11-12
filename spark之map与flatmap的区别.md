spark之map与flatmap的区别
========================

spark的dataframe操作，其中map和flatmap挺绕头的，其实要理解也很简单，只需要记住一下即可：

1. map是对dataframe的每一个row的操作。
2. flatmap是先map，再扁平化。

具体示例我们可以看如下的例子：

    val arr=sc.parallelize(Array("a1","b2","c3"))

使用map：

    arr.map(x=>x).foreach(println)

输出结果为:

    c3
    b2
    a1
   
使用flatmap:

    arr.flatMap(x=>x).foreach(println)

输出结果为：

    b
    a
    1
    c
    2
    3
    
这里可以看到。flatMap首先是对元素进行映射，然后扁平化默认会分割成一个一个字母。


