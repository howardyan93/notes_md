spark学习之dataframe.md
===================================
最近重新捡起spark来学习，现在版本更新为2.1.0. 原来的context被修改为session。但是读取csv，xlsx的功能感觉没多大改进，还有的坑还是在那里。比如读取中文的xlsx文件还是会出现行列错乱的情况。所以有些例子里，读取csv首先是作为文本读入，然后再map分割重组为dataframe。

但是因为spark支持pandas的转换，因此可以借助pandas来读取数据，最后转为spark支持的dataframe：

    from pyspark.sql import SparkSession
    import pandas

    sess = SparkSession.builder.getOrCreate()
    df = pandas.read_csv("./sample_movielens_ratings.txt", sep="::")
    rating = sess.createDataFrame(data=df)

原来的代码如下，大家可以体会下：

    from pyspark.sql import SparkSession

    spark = SparkSession.builder.getOrCreate()
    lines = spark.read.text("./sample_movielens_ratings.txt").rdd
    parts = lines.map(lambda row: row.value.split("::"))
    ratingsRDD = parts.map(lambda p: Row(userId=int(p[0]), movieId=int(p[1]),
                                     rating=float(p[2]), timestamp=long(p[3])))
    ratings = spark.createDataFrame(ratingsRDD)

感觉有点蛋疼。。。
    
    
    
    
    

