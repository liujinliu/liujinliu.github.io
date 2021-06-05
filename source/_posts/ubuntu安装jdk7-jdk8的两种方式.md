---
title: ubuntu安装jdk7-jdk8的两种方式
date: 2018-03-29
categories: 运维
---

原文地址：http://www.cnblogs.com/a2211009/p/4265225.html  

ubuntu安装jdk有两种方式

- 通过ppa方式安装
- 通过官网下载源码安装

这里记录采用ppa方式安装的方法

### 1. 添加ppa
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
```
### 2. 安装oracle-java-installer
jdk7
```
sudo apt-get install oracle-java7-installer
```
jdk8
```
sudo apt-get install oracle-java8-installer
```
安装时候会询问是否同意oracle的服务条款，选择yes.   
如果手懒不想自己动手点击，可以加入下面的命令，自动选择同意(这个是看到原文写了，本人没亲自试过)
##### jdk7默认同意条款
```
echo oracle-java7-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections
```
##### jdk8默认同意条款
```
echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections
```

### 3. 设置系统默认jdk
##### jdk7
```
sudo update-java-alternatives -s java-7-oracle
```
##### jdk8
```
sudo update-java-alternatives -s java-8-oracle
```
### 4. 测试jdk是否安装成功
```
java -version
javac -version
```
