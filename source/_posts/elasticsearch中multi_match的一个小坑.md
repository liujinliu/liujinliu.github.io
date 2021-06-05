---
title: elasticsearch中multi_match的一个小坑
date: 2018-01-18
categories: 编程
---

最近在使用elasticsearch的multi_match搜索时候，使用下面的body对一个字段下的所有字段进行递归搜索，但是当这些子字段出现数值类型的时候，就会报异常了，具体讨论可以参考[这里](https://github.com/elastic/elasticsearch/issues/3975)   
解决方法是加入
```
lenient
```
参考下面的

```
{
    "query": {
        "bool": {
            "should": [
                {
                    "match": {
                        "user_id": {
                            "query": "qwe",
                            "boost": 5
                        }
                    }
                },
                {
                    "multi_match": {
                        "query": "qwer",
                        "lenient": "true",  --ignore values that don't fit specific fields
                        "fields": [
                            "device_brand",
                            "device_manufacturer",
                            "user_properties.*" ---表明搜索此字段下的所有子字段
                        ]
                    }
                },
                {
                    "query_string": {
                        "query": "*qwe*",
                        "default_field": "user_id",
                        "boost": 5
                    }
                }
            ]
        }
    },
    "size": 10,
    "sort": ["_score",{"session_id": "asc"}]
```
