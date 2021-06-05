---
title: Cassandra集群搭建(包含监控)
date: 2017-05-18
categories: 运维
---
[cassandra](http://cassandra.apache.org)是当下流行的nosql数据库，基于google big table的理论。
这篇文章主要是记录自己搭建cassandra集群的过程，更重要的是监控系统的搭建过程，对这种基础设施来说，没有监控就是裸奔，裸奔就相当与*定时炸弹*。

### 1. 集群搭建：
system-info: ubuntu16.04
这里采用debian二进制包的方式安装，这一步也可以采用源码。
在这之前，需要安装好java8的环境，可以参考我另外一篇博客[ubuntu安装jdk7/jdk8的两种方式](http://blog.csdn.net/github_25679381/article/details/68063780)
环境安装好之后，执行下面的命令进行安装：
```
echo "deb http://www.apache.org/dist/cassandra/debian 310x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
```
这里的310x可以自行选择，我安装时候官网给的例子是36x
依次执行的下面的命令进行按照
```
curl https://www.apache.org/dist/cassandra/KEYS | sudo apt-key add -
sudo apt-get update
sudo apt-get install cassandra
```
官网同时提到了在执行
```
sudo apt-get update
```
时候[报错的处理办法](http://cassandra.apache.org/doc/latest/getting_started/installing.html)，我当时没遇到什么报错。
使用
```
nodetool status
```
可以查看的节点的启动状态
节点的启动/停止/重启命令是
```
sudo service cassandra start/stop/restart
```

###2.  集群配置
参考上面的步骤初始化两个机器(192.168.1.100,   192.168.1.101)。
分别配置/etc/cassandra/cassandra.yaml进行集群组建。主要涉及下面的几个配置。  

192.168.1.100:
```
seeds: 192.168.1.100
listen_address: 192.168.1.100
rpc_address： 192.168.1.100
```

192.168.1.101:
```
seeds: 192.168.1.100
listen_address: 192.168.1.101
rpc_address： 192.168.1.101
```
配置完之后重启两台机器，集群就配置完成了。

### 3.监控搭建
下面介绍下集群监控系统的搭建，参考了国外的一篇[文章](https://www.pythian.com/blog/monitoring-apache-cassandra-metrics-graphite-grafana/?utm_source=tuicool&utm_medium=referral)，小伙伴也可以直接去看
监控系统由Graphite和Grafana组成，采用cassandra主动上报状态数据给Graphite，然后Graphite作为Grafana的数据源。因为Grafana的展现给漂亮。
#### 1) 配置cassandra向graphite上报状态统计
1.  下载[metrics-graphite-3.1.0.jar](http://search.maven.org/#artifactdetails|com.yammer.metrics|metrics-graphite|3.1.0|jar)，这需要与你使用的cassandra版本一致。
2.   jar包放入/usr/share/cassandra/lib/
3.  在 /etc/cassandra/目录下创建一个上报状态的配置文件metrics_reporter_graphite.yaml，文件名可以自己随意指定

```
graphite:
  -
    period: 30
    timeunit: 'SECONDS'
    prefix: 'cassandra-TestCluster-192.168.1.227'
    hosts:
     - host: 'localhost'
       port: 2003
    predicate:
      color: 'white'
      useQualifiedName: true
      patterns:
        - '^org.apache.cassandra.+'
        - '^jvm.+'
```
4. 将下面的内容加入cassandra-env.sh

```
METRICS_REPORTER_CFG="metrics_reporter_graphite.yaml"
JVM_OPTS="$JVM_OPTS -Dcassandra.metricsReporterConfigFile=$METRICS_REPORTER_CFG"
```
#### 2) 安装配置Graphite

```
## Install Graphite-carbon and Graphite-web
sudo apt-get install graphite-web graphite-carbon
 
## Configure Graphite
sudo vi /etc/graphite/local_settings.py
   TIME_ZONE = 'Asia/Shanghai'
 
## Sync Graphite database
sudo graphite-manage syncdb

## Configure Carbon
sudo vi /etc/default/graphite-carbon
#-- CARBON_CACHE_ENABLED=true
 
sudo vi /etc/carbon/carbon.conf
#-- ENABLE_LOGROTATION = True
 
## Configure Carbon storage schema
sudo vi /etc/carbon/storage-schemas.conf
#-- Add a section called "cassandra" before the last default section "default_1min_for_1day"
    [cassandra]
        pattern = ^cassandra\.
        retentions = 10s:10m,1m:1h,10m:1d
 
## Configure metrics storage aggregation method
sudo cp /usr/share/doc/graphite-carbon/examples/storage-aggregation.conf.example /etc/carbon/storage-aggregation.conf
#-- Make any changes if needed
 
## Start carbon-cache service
sudo service carbon-cache start
```
上面的配置几乎全copy自参考的那篇博客，除了这里没有使用postgresql，而是使用django默认的数据库(sqlite)
#### 3) 安装配置Grafana
grafana这里使用postgresql数据库，首先安装postgresql

```
## Install Postgres SQL database server
sudo apt-get update
sudo apt-get install postgresql libpq-dev python-psycopg2
sudo -u postgres psql
CREATE USER cassmon WITH PASSWORD 'some_password';
CREATE DATABASE grafana WITH OWNER cassmon;
```
安装配置grafana

```
## Install Grafana
sudo apt-get install grafana
 
## Configure Grafana
sudo vi /etc/grafana/grafana.ini
#-- Make the following changes
    [database]
    type = postgres
    host = 127.0.0.1:5432
    name = grafana
    user = cassmon
    password = cassmon

    [security]
    admin_user = admin
    admin_password = admin
    secret_key = xxxxx@xxxxx.com
sudo service grafana start
```
至此需要配的其实已经都配好了,  此时在8000端口就可以看到graphite的展示，然后在grafana里配置data source，就可以在grafana里看统计了。
