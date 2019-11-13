---
title: 搭建在线jupyter notebook
description:  
date: 2019-11-13 23:48  
categories:
- tool   

tags:  
-  jupyter
 
---
> 使用 jupyter notebook 搭建自己的代码实验田，并积累代码片段 。   

> 环境：Centos 7.4  Python 3.7   

### 创建虚拟环境   
1. `pip3 install virtualenv`  
2. 切换到要项目目录，比如`/home/jupyter-notebook`下，执行`virtualenv env`，新建
的虚拟环境就在 env 内，进入 env/bin 目录，`source active` 激活虚拟环境，标志就是
命令行前有一个`(env)`前缀。下面的操作都是在虚拟环境中进行。

### 安装 jupyter
1. 安装 jupyter `pip install jupyter`
2. `jupyter notebook --generate-config` 生成配置，配置文件地址会在显示在界面上。  
3. 找到配置文件并编辑，主要修改一下几项：
```python
c.NotebookApp.allow_root = True # 允许以 root 用户开启
c.NotebookApp.ip = '*' # 允许访问的 ip 地址
c.NotebookApp.notebook_dir = '/home/jupyter-notebook/notebook' # 工作目录
c.NotebookApp.open_browser = False # 默认是开启后就打开浏览器，但在线不需要，因此False
c.NotebookApp.port = **** # 使用的端口号
```
4. 自动配置访问密码 `jupyter notebook password` 根据提示输入密码。
5. 启动：`jupyter notebook`   

至此，简易的在线服务就完毕了，访问：http:ip:port 就可以使用了，但是还不够好。  

### 为了更好
启动：`nohup jupyter notebook > jupyter.log 2>&1 &`   
停止：`ps -aux | grep jupyter` `kill -9 [pid]`   
使用 nginx 反向代理，使用域名进行访问：
```
server
{
    listen 80;
    server_name www.domain.com;
    
    
    location /
    {
    	proxy_pass http://127.0.0.1:[port];

    }
    access_log  /www/wwwlogs/code.freaks.group.log;
    error_log  /www/wwwlogs/code.freaks.group.error.log;
}
```

现在，访问域名就可以拥有自己的代码实验田。平时自己认为很有意思的代码片段也可收藏到
这里。