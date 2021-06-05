---
title: curl命令发送post表单提交
date: 2018-01-18
categories: 工具
---
最近在用post测试一个web服务时候，发现发过去的post参数中的一个 '+'，在服务端接收后都变成了' '(比如明明发送的是 'g+8'，接收到打印出来却是 'g 8'，命令如下：
```
curl -s -d "data=g+8" http://127.0.0.1:8888/xxxx
```
解决方法是先将 'g+8' 进行urlencode，得到 'g%2b8'，然后在用curl去访问
```
curl -s -d "data=g%2b8" http://127.0.0.1:8888/xxxx
```
-s 参数使得curl减少信息输出，不输出错误和进度信息
