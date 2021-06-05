---
title: elasticsearch调优
date: 2018-01-18
categories: 
---
这部分是从同事的wiki上扒下来的，记在这里，作为以后使用的参考
### 分片和副本调优   
 
1. 更多分片使索引能传送到更多的服务器，意味着可以处理更多的文件，而不会降低性能  
2. 更多分片意味着获取特定文档所需的资源量会减少，因为相较于部署更少分片时，存储在单个分片中的文件数量会更少  
3. 更多分片意味着搜索索引时会面临更多问题，因为必须从更多分片中合并结果，使得查询的聚合阶段需要更多的资源
4. 更多副本会增强集群系统的容错性，因为当原始分片不可用时，其副本将代替原始分片发挥作用，只拥有单个副本，集群可能在不丢失数据的情况下遗失分片。当拥有两个副本时，即时丢失了原始分片及其中一个副本，一切工作仍可以很好地继续下去。
5. 更多副本意味着查询吞吐量会增加，因为执行车讯可以使用分片或分片的任一副本

#### 总结 
副本多：

1. 优点：查询速度快，容错性高
2. 缺点：插入速度变慢

分片多：

1. 优点：插入速度快
2. 缺点：查询速度慢

副本最好设置为2份提高容错性和查询速度，分片设置为节点数量的2倍提高插入速度，如果是大批量导入不做查询的话可取消副本

### Merge 吞吐量调优
系统默认是20mb，实际系统性能可能会更好，可适当调整 

```
curl  -XPUT "http://<ip>:9200/_cluster/settings" -d
'{
    "persistent":
    {
        "indices.store.throttle.type": "merge",
        "indices.store.throttle.max_bytes_per_sec": "200mb"
    }
}'

```

### 刷新频率调优
大批量插入时，修改refresh_interval到-1（不刷新）或大于1s的时间，减少shard刷新间隔 

```
curl -XPUT 'http://<ip>:9200/dw-search/_settings' -d '{ 
    "index" : { 
        "refresh_interval" : "-1" 
    } 
}'
```

插入完成后修改回默认值1s 

```
curl -XPUT 'http://10.1.*.*:9200/dw-search/_settings' -d '{ 
    "index" : { 
        "refresh_interval" : "1s" 
    } 
}'
 
```

另外下面的文章列出了很多有用的url 
http://blog.csdn.net/u014351782/article/details/51207650
其中关闭index这个方法在一次故障处理中起了很大的作用，我们发现系统一直报'too many open files'，然后找到elasticsearch进程，发现确实打开了很多句柄，然后调用 

```
curl http://localhost:9200/_nodes/process\?pretty
```

发现最大句柄数已经是65530 

```
"max_file_descriptors" : 65530,
```
果然重启之后，句柄数很快还是飙到了65530，通过关闭index，句柄数得到了显著减少。

本篇博客还参考下面的博客：
http://www.wklken.me/posts/2015/05/23/elasticsearch-issues.html
