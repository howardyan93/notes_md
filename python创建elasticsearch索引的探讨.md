python创建elasticsearch索引的探讨
==============================
以前用elasticsearch都是先create index，然后发送数据进去。但是最近有个需求，要自动创建索引，所以就只能使用elasticsearch的template功能。

直接上代码：

    from elasticsearch import Elasticsearch
    es = Elasticsearch()

    t1 = {
        u'aliases': {},
        u'mappings': {
            u'_default_': {
                u'_source': {
                    u'enabled': False
                },
                u'properties': {
                    "title": {
                        "type": "text",
                        "analyzer": "ik_max_word",
                        "search_analyzer": "ik_max_word",
                    },
                }
            }
        },
        u'order': 0,
        u'settings': {u'index': {u'number_of_shards': u'1'}},
        u'template': u'te*'
    }


    es.indices.put_template(name='test', body=t1, create=True)

这个时候显示：

    {u'acknowledged': True}

就表示template创建成功了。以后只要是创建以te为开头的index，都会自动套用上面的模板设定，比如：

    body = {'title': u'我到河北省来'}
    es.create(index='test_001', doc_type='doc', body=body, id=1)

显示成功后我们再查看test_001的mapping：

    es.indices.get_mapping(index='test_001')

显示：

    {
        u'test_001': {
            u'mappings': {
                u'_default_': {
                    u'_source': {
                        u'enabled': False
                    },
                    u'properties': {
                        u'title': {
                            u'analyzer': u'ik_max_word',
                            u'type': u'text'
                        }
                    }
                },
                u'doc': {
                    u'_source': {
                        u'enabled': False
                    },
                    u'properties': {
                        u'title': {
                            u'analyzer': u'ik_max_word',
                            u'type': u'text'
                        }
                    }
                }
            }
        }
    }
    
说明索引的确是按template自动创建了。

更深入的探讨，如果我们创建一个新的模板，通配test\*的模型的话，对于test_001这样的索引，又会怎样呢？

    t2 = {
        u'aliases': {},
        u'mappings': {
            u'_default_': {
                u'_source': {
                    u'enabled': False
                },
                u'properties': {
                    u"author": {u"type": u'keyword'}
                }
            }
        },
        u'order': 1,
        u'settings': {u'index': {u'number_of_shards': u'1'}},
        u'template': u'test*'
    }
    es.indices.put_template(name='test_sub', body=t2, create=True)

这个时候再创建一个新的索引test_002, 它的mapping会这样：

    {
        u'test_003': {
            u'mappings': {
                u'_default_': {
                    u'_source': {
                        u'enabled': False
                    },
                    u'properties': {
                        u'author': {
                            u'type': u'keyword'
                        },
                        u'title': {
                            u'analyzer': u'ik_max_word',
                            u'type': u'text'
                        }
                    }
                },
                u'doc': {
                    u'_source': {
                        u'enabled': False
                    },
                    u'properties': {
                        u'author': {
                            u'type': u'keyword'
                        },
                        u'title': {
                            u'analyzer': u'ik_max_word',
                            u'type': u'text'
                        }
                    }
                }
            }
        }
    }
    
说明test和test_sub的模板同时应用到了test_002这个索引上了。但是这是没有冲突的情况，如果我们修改test_sub的配置，添加跟test重名的title字段，但是配置不同的话，又会怎么样呢？

    t3 = {
        u'aliases': {},
        u'mappings': {
            u'_default_': {
                u'_source': {
                    u'enabled': False
                },
                u'properties': {
                    u"title": {u"type": u'keyword'}
                }
            }
        },
        u'order': 1,
        u'settings': {u'index': {u'number_of_shards': u'1'}},
        u'template': u'test*'
    }
    es.indices.put_template(name='test_sub', body=t3)
   
然后创建新的索引test_003, 直接报错：

    TransportError(400, u'mapper_parsing_exception', u'Mapping definition for [title] has unsupported parameters:  [search_analyzer : ik_max_word] [analyzer : ik_max_word]')

看来似乎es还支持子模板字段重名的情况。。。我用es的版本是5.4.3，不知道将来会不会有改动。

     






    

    


