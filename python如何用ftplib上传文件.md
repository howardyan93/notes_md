python如何用ftplib上传文件
==============================
python的ftplib的文档里只有如何下载，没有教如何上传，自己对ftp命令也不熟悉。然手贱了一下国内的解决方法，有个哥们的答案是这样的：

    ftp.storbinary(filename, file_handler)

我勒个去，文档里面明明要求是命令，只传文件名行不通好么？

直接搜索stackoverflow，立马有答案:

    ftp.storbinary('STOR %s' % filename, file_handler)
