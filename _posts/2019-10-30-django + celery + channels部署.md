---
title: django + celery + channels部署  
description:  
date: 2019-10-30 1:34  
categories:
- django   
tags:  
- python  
- celery
- channel
- daphne
 
---
最近尝试已一些库和框架，将平常自己写的功能整合到一起。
整体结构如下图：  
![结构图](http://qiniu2.freaks.group/站点架构.jpg)   
橙色框是主要配置点
uwsgi 服务器的配置大都熟悉，

#### daphne 和 uWSGI 
1. 安装 supervisor，`Centos` 下直接 `yum install supervisor` 
2. 其配置文件路径：`/etc/supervisord.conf` ，可以查看详细的配置。一般在配置的最后一行有一个路径
指向我们的配置路径，类似于：files = supervisord.d/*.ini，那我们就可以进入到此路径内定义程序。  
3. 下面是示例：
```yaml
# last_site_websocket.ini
[program:last_site_websocket]
command=/usr/local/python3/bin/daphne -b 0.0.0.0 -p 9001 -u /tmp/daphne.sock backend_for_last_site.asgi:application
directory=/www/wwwroot/api.freaks.group/global/backend_for_last_site/
stdout_logfile=/www/wwwroot/api.freaks.group/websocket_out.log
stderr_logfile=/www/wwwroot/api.freaks.group/websocket_err.log
autostart=true
autorestart=true
user=root
startsecs=10
```
```yaml
# uwsgi.ini 文件命名尽量和 app一致
[program:uwsgi]
command=uwsgi3 --ini /www/wwwroot/api.freaks.group/global/backend_for_last_site/uwsgi.ini
directory=/www/wwwroot/api.freaks.group/global/backend_for_last_site
stdout_logfile=/www/wwwroot/api.freaks.group/uwsgi_out.log
stderr_logfile=/www/wwwroot/api.freaks.group/uwsgi_err.log
autostart=true
autorestart=true
user=root
startsecs=3
```
4. 使用方式：`supervisorctl [stop/start/restart] program_name`

#### celery 的部署
celery 部署采用的是 Generic init-scripts 方式 [文档](http://docs.celeryproject.org/en/latest/userguide/daemonizing.html#generic-init-scripts)
但是服务器上是没有 celeryd 和 celerybeat 这两个脚本的，我们需要从[这里](https://github.com/celery/celery/tree/master/extra/generic-init.d/)
将这两文文件复制到 `/etc/init.d/`路径下。然后再按照文档的说明进行配置就可以了。  

#### nginx 
主要就是 websocket 的转发问题，这个和 uwsgi 相似，添加以下配置。
```
location /ws {
        proxy_pass http://127.0.0.1:9001; # 端口和daphne的保持一致

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_redirect     off;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
        proxy_read_timeout  36000s;
        proxy_send_timeout  36000s;
    }
```

