---
title: mysql插入太多数据导致MySQL-server-has-gone-away
date: 2018-04-23
categories: database
---
最近在一次开发过程中，将数据写入mysql时候总是报
```
MySQL server has gone away
```
的错误，因为mysql就是本地启动的，而且从插入到报错整个过程时间很短，因此不可能是连接方面的timeout一类的问题。最后通过网上搜索，看到了遇到类似问题的[帖子](http://blog.csdn.net/fdipzone/article/details/51974165)，参考作者的做法，问题得到了解决。   
 <!--more-->  
原因是max_allowed_packet参数导致的。按照官方解释增大这个参数可以使得从client到server端传递大数据时，系统能够分配更多的扩展内存来处理。   
首先查看mysql max_allowed_packet的值
```
mysql> show global variables like 'max_allowed_packet';
+--------------------+---------+
| Variable_name      | Value   |
+--------------------+---------+
| max_allowed_packet | 4194304 |
+--------------------+---------+
1 row in set (0.00 sec)
```
可以看到默认是4M，我的数据达到了5M多，难怪报错。修改如下
```
mysql> set global max_allowed_packet=268435456;
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like 'max_allowed_packet';
+--------------------+-----------+
| Variable_name      | Value     |
+--------------------+-----------+
| max_allowed_packet | 268435456 |
+--------------------+-----------+
1 row in set (0.00 sec)
```
至此问题得到解决。另外如果重启mysql，这里的设置会被还原。如果想重启后不还原，可以打开 my.cnf 文件，添加
```
max_allowed_packet = 256M
```
