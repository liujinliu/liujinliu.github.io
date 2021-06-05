---
title: mysql-python安装时EnvironmentError-mysql_config-not-found
date: 2018-01-18
categories: database
---
转自：http://www.cnblogs.com/xiazh/archive/2012/12/12/2814289.html
在安装 mysql-python时，会出现：
```
sh: mysql_config: not found
Traceback (most recent call last):
  File "setup.py", line 15, in <module>
    metadata, options = get_config()
  File "/home/zhxia/apps/source/MySQL-python-1.2.3/setup_posix.py", line 43, in get_config
    libs = mysql_config("libs_r")
  File "/home/zhxia/apps/source/MySQL-python-1.2.3/setup_posix.py", line 24, in mysql_config
    raise EnvironmentError("%s not found" % (mysql_config.path,))
EnvironmentError: mysql_config not found
```

解决方法如下
```
sudo apt-get install libmysqlclient-dev
sudo updatedb
locate mysql_config
```
mysql_config的位置为：/usr/bin/mysql_config
通常情况下这就ok了，反正我的系统是ubuntu16.04这样做之后就好了。  

旧一点的系统据说还需要按下面的方式再处理下，仅供参考： 
在mysql-python源码包下找到：setup_posix.py 文件，然后找到文件中的 mysql_config.path 将其值改为：/usr/bin/mysql_config,然后 sudo python setup.py install ,就ok
