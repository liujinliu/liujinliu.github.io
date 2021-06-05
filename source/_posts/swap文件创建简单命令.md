---
title: swap文件创建简单命令
date: 2017-06-28
categories: 运维
---
创建一个10G的swap文件，放在home下面，给当前用户和root使用(由文件权限决定) 
```
cd ~/
dd if=/dev/zero of=swapfile bs=1G count=10
chmod 600 swapfile
mkswap swapfile
sudo swapon swapfile
```
