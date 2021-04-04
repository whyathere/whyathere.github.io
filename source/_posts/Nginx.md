---
title: Nginx
date: 2018-09-10 18:27:21
tags: Nginx
categories: 服务器
---

### 启动Nginx

#### nginx启动文件的目录

/usr/local/bin

然后运行命令

```shell
sudo ./nginx
```

重启 sudo ./nginx -s reload 

#### Nginx的安装目录

/usr/local/etc/nginx

Nginx.conf

```properties
    server {
        listen       800;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /Users/mac/workspace/web/;
            index  index.html index.htm;
        }
```

