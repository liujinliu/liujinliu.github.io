---
title: rabbitmq的简单管理命令笔记
date: 2018-01-18
categories: 运维
---
#### rabbitmq最新版本在外部的访问权限上进行了进一步的控制，其中默认情况下，guest用户只能通过本地loopback端口访问
为了在外部对rabbitmq进行连接和访问，需要新增用户，对用到的命令进行简单记录
```
rabbitmqctl add_user <username> <userpass>
rabbitmqctl add_vhost <path>
rabbitmqctl set_user_tags <username> administrator
rabbitmqctl set_permissions -p <path> <username> ".*" ".*" ".*"
```
#### rabbitmq简单状态查询命令
```
rabbitmqctl list_connections   ---用于查看当前的连接
rabbitmqctl list_queues        ---会列出所有队列名称，后边可能还会带着这个队列当前消息数
rabbitmqctl status             ---查看当前队列信息
```
#### rabbitmq恢复出厂设置命令
```
rabbitmqctl stop_app
rabbitmqctl reset/force_reset
rabbitmqctl start_app
```
#### rabbitmq清除队列里的消息
```
rabbitmqctl -p ${vhost-name} purge_queue ${queue-name}
```
