logstash应用配置把玩
==============================
ELK平台随着大数据/互联网的兴起收到了越来越多的关注。其中Elasticsearch和Kibana自不必说。就连logstash也随着发展，由原来定位于日志收集整理的小工具，变成了一个万能的转换管道了。大部分功能在ETL应用中也有奇效。这里就介绍一下logstash在ETL中的应用。

logstash的概念源于unix的管道，其配置主要也分为三个部分: input, filter, output。

Logstah的配置模板如下：

    input {}
    filter {}
    output {
        stdout { codec => rubydebug }
    }

其中ouput中会设置stdout输出到命令行，方便debug.

从最简单的示例开始，比如将TSV数据倒到Kafka, 我们以最有名的iris花数据为例。

iris数据样例如下：

    5.1,3.5,1.4,0.2,setosa
    5.1,3.5,1.4,0.2,setosa
    ......
    7.0,3.2,4.7,1.4,versicolor
    6.4,3.2,4.5,1.5,versicolor     
    ......
    6.3,3.3,6.0,2.5,virginica
    5.8,2.7,5.1,1.9,virginica

导入配置如下：

    input {
        file {
            path => "/path/*.tsv"
        }
    }
    filter {
       csv {
         separator => ","
         columns => ["a1", "a2", "a3", "a4", "a5"]
       }
    }
    output {
        stdout { codec => rubydebug }
        kafka {
            codec => plain {
               format => "%{a1}|++|%{a2}|++|%{a3}|++|%{a4}|++|%{a5}"
            }
            topic_id => "test_topic"
        }
    }    

input首先从指定的路径读入tsv文件。logstash在读完文件后会记录已读取过的文件以及上次导入的位置，避免重复入库。

filter对数据进行整理，首先通过逗号分割符将数据分割，同时将字段分别映射为a1-a5。

ouput在对数据进行整理，将数据重新拼接后，输出到kafka的test_topic。这里没有设置kafka的主机，因此默认是localhost:9092

在实际应用中，还有除了输出的倒来倒去。还需要对数据进行格式的处理。如时间戳转字符串，ip转地理位置信息。前者可以通过ruby函数来转换，后者需要通过映射表来进行映射。在上面的例子里，我们要对不同种类的花设置不同的价格的话可以这样配置：

    filter {
        csv {
            separator => ","
            columns => ["a1", "a2", "a3", "a4", "a5"]
        }
        translate {
            field => "a5"
            destination => "price"
            dictionary => {
                "virginica" => "10$"
                "setosa" => "20$"
                "versicolor" => "30$"
            }
        }
    }

命令行中就会显示出price这个字段。

如果映射表很大的时候，写到配置文件里就显得非常不合适，logstash同时提供了从其他文件或者数据库中获取映射字段的方式，以数据库为例：

    filter {
        csv {
            separator => ","
            columns => ["a1", "a2", "a3", "a4", "a5"]
        }
        jdbc_streaming {
            jdbc_driver_library => "/path/logstash_jdbc/sqljdbc4-4.0.2206.100.jar"
            jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
            jdbc_connection_string => "jdbc:sqlserver://xx.xx.xx.xx\test:3433;databaseName=test_db"
            jdbc_user => "username"
            jdbc_password => "password"
            statement => "select price from dbo.test WHERE FactoryCode = :a5"
            parameters => { "a5" => "a5"}
            target => "new"
        }
        split {
            field => "new"
        }
    }

通过配置jdbc去访问sqlserver服务器，然后通过查询a5得到对应的映射字段。由于查询返回的数据是一个列表，这里通过split功能将列表分割成一个一个独立的数据，达到跟以前一样的效果。




