---
title: 记一次inodes占用过多问题的解决
date: 2018-01-18
categories: 操作系统
---
最近收到某台服务器告警，inodes使用过高，解决过程如下  
首先在服务器上执行下面的命令查看哪个目录下inodes使用过高
```
[root@vm]# df -i
Filesystem              Inodes  IUsed     IFree IUse% Mounted on
/dev/mapper/VGSYS-lv_root
                        655360 101872    553488   16% /
tmpfs                  2041469      1   2041468    1% /dev/shm
/dev/vda1                51200     38     51162    1% /boot
/dev/mapper/VGSYS-lv_var
                        655360 569533     85827   87% /var
/dev/mapper/VGSYS-lv_srv
                     104644608   1727 104642881    1% /srv
/dev/mapper/VGSYS-lv_vfs
                      52428800      7  52428793    1% /srv/docker/vfs
/dev/mapper/VGSYS-lv_log
                      31457280    207  31457073    1% /var/log
```
可以发现/var目录下inodes使用最大，使用下面的脚本进一步查找
```
[root@vm]# for i in /var/*; do echo $i; find $i | wc -l; done
/var/account
2
/var/cache
98
......
/var/spool
587003
/var/tmp
1
```
通过这个方法，最终发现是"/var/spool/postfix/maildrop"目录下小文件过多。  
通过上网搜索，原因如下： 
```
是由于linux在执行cron时，会将cron执行脚本中的output和warning信息，都会以邮件的形式发送Cron所有者，  
而由于客户环境中的sendmail和postfix没有正常运行，导致邮件发送不成功，全部小文件堆积在了maildrop目录 
下面，而且没有自动清理转换的机制，所以此目录堆积了大量的文件
```
进入"/var/spool/postfix/maildrop"路径，使用
```
ls | xargs -n 10 rm -rf 
```
将文件清楚，问题得以恢复。

### 根本解决方法
```
vi /etc/crontab
将MAILTO=root替换MAILTO=""，然后service crond restart即可。如不行crontab -e 第一行增加MAILTO=""
```
