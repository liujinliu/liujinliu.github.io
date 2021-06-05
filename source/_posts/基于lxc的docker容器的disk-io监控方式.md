---
title: 基于lxc的docker容器的disk-io监控方式
date: 2018-01-18
categories: 运维
---
以一台测试机上的容器d-mcl-30_struc_test-n-2为例，容器id为373200daed7b，通过```docker inspec```t可以看到如下信息： 
![20161110190828780.png](https://i.loli.net/2021/06/05/DXRsuwyMSJnm6fP.png)

图中高亮出来的部分是我们从宿主机映射到容器内的一个路径，下面我们来看这个路径对应的设备
![20161110190908546.png](https://i.loli.net/2021/06/05/QLy4uOPDrjHnRhS.png)

通过下面的方法查看这个设备的真实路径(因为可能是一个软链接)
![20161110191023781.png](https://i.loli.net/2021/06/05/niowJ1trVLQWPG9.png)

通过下面的方法查找挂载设备的设备号：
![20161110191111069.png](https://i.loli.net/2021/06/05/TRmSfF4HzqIDPJK.png)

容器挂载路径disk设备的io可以通过监控下面的文件得到
![20161110191208601.png](https://i.loli.net/2021/06/05/ynXWOQ8TiY4ISBt.png)

这个文件记录的是当前这个容器在这个设备的写入/读取字节数，通过对这个文件的监控，取两次监控数值的差，然后除以取样间隔时长得到disk-iops  

注意
如果通过nsenter命令进入一个容器采用dd命令进行磁盘读写测试，测试产生的读写是不会反应在这里说的文件上的。需要通过docker attach命令进入才可以
