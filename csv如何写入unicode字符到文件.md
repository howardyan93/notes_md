csv如何写入unicode字符到文件
===================================
python使用csv库写unicode字符到文件会抛出unicode转换异常，比如在stackoverflow上就有哥们遇到过这样的

    UnicodeEncodeError: 'ascii' codec can't encode character u'\xef' in position 0: ordinal not in range(128)

后面的解决方法很多，比如使用pip上的另一个csv模块，但是要额外安装很蛋疼。

问题的关键在于打开python2.7打开文件默认是ascii的，所以写入的时候有问题。直接用codecs以utf-8格式打开一个文件，再写入即可：

    import codecs
    with codecs.open(filename, "wb", "utf-8") as file:
        writer = csv.DictWriter(file, fieldnames=header)
        writer.writeheader()
        for row in rows:
            writer.writerow(row)

