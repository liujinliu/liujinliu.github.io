---
title: jq工具的简单使用
date: 2018-01-18
categories: 工具
---
最近因为工作需要，同事给我一个json文件，整个文件3M，全在一行，我去，简直崩溃。  
使用[jq](https://stedolan.github.io/jq/)工具很好的解决了这个问题。
### 1. 安装jq
```
sudo apt-get install jq
```

### 2. 在vim中的使用
最后一行模式下
```
:%!jq '.'
```
需要注意的是这样操作原文件就直接被改变了，所以最好提前备份下。
