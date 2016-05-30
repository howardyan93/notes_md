python中如何以可读的方式查看json
==================================
最近学习抓取网页的数据，有些返回的json数据很难看。print命令挤成了一坨。

如果是在win平台下，有很多工具可以对json进行格式转换。但是作为在linux平台跑python的人士，还是要通过python来自己搞定。

json库本身就具有对json格式化输出的功能，这还是水木ID为hrpenf的大神教我的:

    import json
    print json.dumps(dumps({'4': 5, '6': 7}, sort_keys=True, indent=4, separators=(',', ': '))


