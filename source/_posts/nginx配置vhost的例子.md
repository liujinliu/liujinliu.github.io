---
title: nginx配置vhost的例子
date: 2018-01-18
categories: 运维
---
nginx下面几个配置vhost的例子 
php
```
server {
server_name stage.localhost;
listen 80 ;
        root /data;
        index index.html index.htm index.php;
location ~ .php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME  /data$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
静态文件
```
server {
server_name sms.fake;
listen 80 ;
        root /data/fakesms;
        index mobile1.html mobile1b.html;
}
```
其他语言开发的server服务
```
server {
    listen   80;
    server_name localhost.pythonserver;

    access_log /var/log/nginx/pythonserver-access.log ;
    error_log /var/log/nginx/pythonserver-error.log ;
    charset utf-8;
    client_max_body_size    100m;
    client_body_timeout     60;      
    location /exam0/ {
        proxy_pass http://127.0.0.1:8889;
    }
    location /exam1/ {
        proxy_pass http://127.0.0.1:8880;
    }
    location /exam2/ {
        proxy_pass http://127.0.0.1:8881;
    }
}
```
带upstream的写法例子：
```
upstream rd_servers {
  server 127.0.0.1:5000;
}

server{
  server_tokens off;
  listen 80;
  server_name redash.xxxxx.com;
  access_log /var/log/nginx/rd.access.log;

  gzip on;
  gzip_types *;
  gzip_proxied any;

  location / {
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass       http://rd_servers;
  }
}
```
需要注意的一点是，nginx默认的安装配置路径在/etc/nginx下面，在nginx.conf里面一般可以看到下面一行内容
```
...
include /etc/nginx/conf.d/*.conf;
...
```
因此可以在conf.d下面灵活的增加自己的配置文件。
```
listen {port}
```
这种配置是可以在多个配置文件出现的，但要注意的是同一个port，那么不同的配置文件一定要有不同的
```
server_name
```
