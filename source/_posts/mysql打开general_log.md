---
title: mysql打开general_log
date: 2018-01-18
categories: database
---
最近在接收同时开发完的代码进行调试时候，一个事务执行过程没有报错，但是结果就是无法往数据库插入数据，最后打开general_log才发现是因为在代码执行过程，每次调用sdk接口进行sql执行时候都是重新连数据库，所以最后的commit相当于没有任何提交
方法如下：
```
#>: mysql  -uroot -p{passwd} -P3306
# 查看当前general_log是否打开(OFF表明是关闭的)
mysql> show variables LIKE '%general%';
+------------------+-----------------------------------------------+
| Variable_name    | Value                                         |
+------------------+-----------------------------------------------+
| general_log      | OFF                                           |
| general_log_file | /srv/rds_cluster/mysql/xxxxxxxx.log |
+------------------+-----------------------------------------------+
2 rows in set (0.00 sec)

# 打开general_log
mysql> set global general_log=ON;
Query OK, 0 rows affected (0.00 sec)

mysql> quit
Bye
```
可以执行一些语句然后再查看下general_log
```
mysql> select * from user1;
mysql> quit
Bye
# 查看general_log
-bash-4.1# cat /srv/rds_cluster/mysql/xxxxxxxx.log 
/opt/letv/mcluster/root/usr/libexec/mysqld, Version: 5.5.34-log (MySQL Galera Cluster (GPL), wsrep_23.7.6.rXXXX). started with:
Tcp port: 3306  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
161117 11:12:04   944 Quit
161117 11:12:11   945 Connect   root@localhost on 
                  945 Query     select @@version_comment limit 1
161117 11:12:20   945 Query     SELECT DATABASE()
                  945 Init DB   linzhanbo
                  945 Query     show databases
                  945 Query     show tables
                  945 Field List        ___user1_old 
                  945 Field List        __user1_old 
                  945 Field List        _user1_old 
                  945 Field List        user1 
                  945 Field List        userinfo 
161117 11:12:25   945 Query     show tables
161117 11:12:33   945 Query     select * from user1
161117 11:12:37   945 Quit
```
下面就是我打开general_log之后定位问题发现的日志，可以看出来，每次执行sql都重新连接了mysql，这样事务最终没有执行成功也就比较好定位了。
```
161117 11:22:15   947 Connect   root@127.0.0.1 on xxxxxx
                  947 Query     SET NAMES utf8
                  947 Query     set autocommit=0
                  947 Query     insert into user1(ID,age) values(0,23)
                  948 Connect   root@127.0.0.1 on xxxxxx
                  948 Query     SET NAMES utf8
                  948 Query     set autocommit=0
                  947 Quit
                  949 Connect   root@127.0.0.1 on xxxxxx
                  949 Query     SET NAMES utf8
                  949 Query     set autocommit=0
                  948 Quit
                  949 Quit
                  950 Connect   root@127.0.0.1 on xxxxxx
                  950 Query     SET NAMES utf8
                  950 Query     set autocommit=0
                  950 Query     insert into user1(ID,age) values(1,24)
                  951 Connect   root@127.0.0.1 on xxxxxx
                  951 Query     SET NAMES utf8
                  951 Query     set autocommit=0
                  950 Quit
                  952 Connect   root@127.0.0.1 on xxxxxx
                  952 Query     SET NAMES utf8
                  952 Query     set autocommit=0
                  951 Quit
                  952 Quit
                  953 Connect   root@127.0.0.1 on xxxxxx
                  953 Query     SET NAMES utf8
                  953 Query     set autocommit=0
                  953 Query     commit
                  953 Quit
```
另外需要说明一点的是create表和drop表的操作是不需要commit的。
