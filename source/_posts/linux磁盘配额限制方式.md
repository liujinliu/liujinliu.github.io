---
title: linux磁盘配额限制方式
date: 2018-01-18
categories: 操作系统
---
最近研究docker的磁盘配额限制，貌似官方最新版本的docker已经支持，底层用的是linux的quota，于是自己对quota进行了简单的尝试，记录于此。本文将演示如何创建一个gquota的用户组，并对这个用户组下的用户进行磁盘配额限制。 
#### 查看内核是否支持
```
# grep CONFIG_QUOTA /boot/config-[version]
CONFIG_QUOTA=y
CONFIG_QUOTA_NETLINK_INTERFACE=y
# CONFIG_QUOTA_DEBUG is not set
CONFIG_QUOTA_TREE=y
CONFIG_QUOTACTL=y
```
CONFIG_QUOTA和CONFIG_QUOTACTL两个y，恭喜，你的内核支持quota，否则就要升级或是重新编译内核了，貌似2.4以上的版本内核都支持
#### 修改内核fstab，对根目录开启磁盘配额限制
```
# vim /etc/fstab
```
![这里写图片描述](http://ljlimg.aimag.top/blog/images/20161027115636528)
标出来的地方就是需要新增的地方，这个表示是对根目录进行磁盘配额限制，当然，也可以加在其他行，则是对其他的目录进行磁盘配额限制。
#### 查看目录挂载位置
```
# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        16G   11G  4.4G  71% /
tmpfs           938M   72K  938M   1% /dev/shm
/dev/sda1       190M   32M  148M  18% /boot
```
#### 重新挂载根目录分区，内核重新读取/etc/fstab文件
```
# mount -o remount /
# mount | grep quota
/dev/sda3 on / type ext4 (rw,usrquota,grpquota)
```
#### 通过quotacheck命令在根目录下生成quota配置文件
```
# quotacheck -cugm /dev/sda3
# ll / | grep quota
-rw-------.   1 root root   8192 Oct 27 12:01 aquota.group
-rw-------.   1 root root   7168 Oct 27 11:43 aquota.user
```
#### 启动磁盘配额
```
# quotaon /dev/sda3
```
#### 创建一个用户组gquota，用来测试
```
# groupadd gquota
```
#### 配置对用户组gquota的磁盘配额限制
```
# edquota -g gquota
```
![这里写图片描述](http://ljlimg.aimag.top/blog/images/20161027120951128)
设置用户组gquota的软限制为480M，硬限制为500M（用户超过软限制会得到报警，超不过硬限制）
#### 新建用户并加入gquota用户组
```
# useradd -m temp
# passwd temp
# usermod -g gquota temp
# su - temp
```
#### 模拟大文件写入
```
# dd if=/dev/zero of=/home/temp/file1 bs=1M count=450
```
![这里写图片描述](http://ljlimg.aimag.top/blog/images/20161027134117449)
大小没有超过软限制，没有告警
再次写入
```
# dd if=/dev/zero of=/home/temp/file3 bs=1M count=40
```
![这里写图片描述](http://ljlimg.aimag.top/blog/images/20161027134601486)
此时已经出现告警
再次写入
```
# dd if=/dev/zero of=/home/temp/file3 bs=1M count=40
```
![这里写图片描述](http://ljlimg.aimag.top/blog/images/20161027134956569)
显示写入失败
#### 查看磁盘分配使用情况
![这里写图片描述](http://ljlimg.aimag.top/blog/images/20161027135032847)
### 说明
需要说明的一点是，对组的限制指的是这个组下面的所有用户加起来使用的磁盘总额，假设有个temp2用户也加入了gquota这个组，那么如果temp用户已经写入了400M，那么留给temp2用户的软限制则只剩80M，磁盘配额限制是可以针对单个用户的，即执行下面的命令，依然以新建一个temp用户为例，temp用户不用加入gquota组
```
[root@centos66-3 ~]# useradd -m temp
[root@centos66-3 ~]# passwd temp
[root@centos66-3 ~]# edquota -u temp
```
![这里写图片描述](http://ljlimg.aimag.top/blog/images/20161027140343586)
配置temp用户的磁盘配额软限制和硬限制依然可以采用上面的方式进行写入测试。
