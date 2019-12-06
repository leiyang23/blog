---
title: Telegraf+InfluxDB+Grafana构建监控平台   
description:  
date: 2019-12-06 23:42  
categories:  
- golang
- 运维      

tags:  
- 
 
---

## Telegraf+InfluxDB+Grafana构建监控平台 
> `influxdb` 是一款开源的时序数据库 ，使用 go 编写。    
> 
>`telegraf` 是一款系统和服务的统计数据插件，可以讲数据插入到 InfluxDB，也是使用 go 编写，和 influxdb 同属一家公司。    
>
>`grafana` 构建监控平台是一个开源指标分析和可视化套件，常用于可视化基础设施的性能数据和应用程序分析的时间序列数据。  
>
### 安装 Telegraf 和 InfluxDB
```shell script
cat <<EOF | sudo tee /etc/yum.repos.d/influxdata.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```
`sudo yum install influxdb` `sudo yum install telegraf`     
对于CentOS 7以上的版本：支持`systemctl start|status|stop|restart [influxdb|telegraf]`    
telegraf 启动后会自动关联 influxdb，并建立 一个 telegraf 的数据库。      

查看 influxdb 配置：`influxd config`     
influxdb [中文文档](https://jasper-zhang1.gitbooks.io/influxdb/content/Introduction/installation.html)      
telegraf 的详细配置说明参考此[博文](https://www.cnblogs.com/panjunbai/p/9568928.html)   

### 安装 Grafana
```shell script
cat <<EOF | sudo tee /etc/yum.repos.d/grafana.repo
[grafana]
name=grafana
baseurl=https://mirrors.tuna.tsinghua.edu.cn/grafana/yum/rpm
repo_gpgcheck=0
enabled=1
gpgcheck=0
EOF
```  
`sudo yum makecache &&  yum install grafana`    
支持`systemctl start|status|stop|restart grafana-server`      
默认开启后 服务地址`http://localhost:3000` ,默认账户：`admin/admin`。       
首先配置数据源，如图：   
![数据源](http://qiniu2.freaks.group/grafana%20%E6%95%B0%E6%8D%AE%E6%BA%90%E9%85%8D%E7%BD%AE.png)
数据库默认不作权限认证，`InfluxDB Details` 填入数据库名就行。

然后是导入配置：
![看配置板导入](http://qiniu2.freaks.group/grafana%20%E7%9C%8B%E6%9D%BF%E5%AF%BC%E5%85%A5.png)     
一个针对 telegraf 的配置地址：`https://grafana.com/grafana/dashboards/928`。   

> 参考文档：https://juejin.im/post/5b4568c851882519790c72f3
    


