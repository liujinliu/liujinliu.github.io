---
title: 记python中json反序列化的一次问题解决
date: 2018-01-18
categories: 编程
---
最新在tornado接收body为json数据时，进行json的反序列化时候遇到一个问题，输入如下：
```
 body='{\n\t"pts":[\n\t\t{\n\t\t\t"ddlSqls":"create table user1(ID Varchar(20) NOT NULL, name Varchar(20))",\n\t\t\t"type":"CREATE"\n\t\t},\n\t\t{\n\t\t\t"tbName":"user1",\n\t\t\t"ddlSqls":"add column age int,drop column name",\n\t\t\t"type":"ALERT"\n\t\t},\n\t\t]\n}'
```
这里直接调用
```
json.loads(body, encoding='utf-8')
```
会产生如下异常：
```
HTTPRequest(protocol='http', host='10.185.81.201:8888', method='POST', uri='/db/chenwenquan/ddl/batch', version='HTTP/1.1', remote_ip='10.75.139.125', body='{\n\t"pts":[\n\t\t{\n\t\t\t"ddlSqls":"create table user1(ID Varchar(20) NOT NULL, name Varchar(20))",\n\t\t\t"type":"CREATE"\n\t\t},\n\t\t{\n\t\t\t"tbName":"user1",\n\t\t\t"ddlSqls":"add column age int,drop column name",\n\t\t\t"type":"ALERT"\n\t\t},\n\t\t]\n}', headers={'Origin': 'chrome-extension://fhbjgbiflinjbdggehcddcbncdddomop', 'Content-Length': '222', 'Accept-Language': 'zh-CN,zh;q=0.8', 'Accept-Encoding': 'gzip, deflate', 'Host': '10.185.81.201:8888', 'Accept': '*/*', 'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36', 'Connection': 'keep-alive', 'Cache-Control': 'no-cache', 'Content-Type': 'application/json', 'Postman-Token': '65530e42-6a55-ad55-f91c-1ee81808cacc'})
Traceback (most recent call last):
  File "/usr/lib/python2.6/site-packages/tornado/web.py", line 1073, in wrapper
    return method(self, *args, **kwargs)
  File "/usr/lib/python2.6/site-packages/tornado/gen.py", line 104, in wrapper
    gen = func(*args, **kwargs)
  File "/opt/letv/mcluster-manager/api/handlers/database.py", line 45, in post
    body = json.loads(self.request.body, encoding='utf-8')
  File "/usr/lib64/python2.6/json/__init__.py", line 318, in loads
    return cls(encoding=encoding, **kw).decode(s)
  File "/usr/lib64/python2.6/json/decoder.py", line 319, in decode
    obj, end = self.raw_decode(s, idx=_w(s, 0).end())
  File "/usr/lib64/python2.6/json/decoder.py", line 336, in raw_decode
    obj, end = self._scanner.iterscan(s, **kw).next()
  File "/usr/lib64/python2.6/json/scanner.py", line 55, in iterscan
    rval, next_pos = action(m, context)
  File "/usr/lib64/python2.6/json/decoder.py", line 183, in JSONObject
    value, end = iterscan(s, idx=end, context=context).next()
  File "/usr/lib64/python2.6/json/scanner.py", line 55, in iterscan
    rval, next_pos = action(m, context)
  File "/usr/lib64/python2.6/json/decoder.py", line 219, in JSONArray
    raise ValueError(errmsg("Expecting object", s, end))
ValueError: Expecting object: line 12 column 3 (char 219)
```
解决方式采用如下的方式
```
json.loads(body, strict=False, encoding='utf-8')
```
