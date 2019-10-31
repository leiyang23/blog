---
title: 利用requests 模拟登陆csdn 
description:  
date: 2019-09-26 18:03  
categories:
- 实例   
- 老博客迁移
tags:  
- python  
 
---
环境：python3.6.1 + lxml4.0.0 + requests2.18.4
>坑一：登陆时请求的网址需要构造，数据在form标签属性里，
坑二：表单数据的提取
坑三：登陆后的跳转，不然无法访问个人主页

```
import requests
from lxml import etree


#设置session
s=requests.Session()

#基础参数
url='https://passport.csdn.net'
headers={'Host':'passport.csdn.net',
'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:55.0) Gecko/20100101 Firefox/55.0',
'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
'Accept-Language':'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
'Referer': 'https://passport.csdn.net/?service=http://write.blog.csdn.net/postedit'
}

#抓取post参数和jsessionid
f=s.get(url,headers=headers).content.decode('utf-8')
p=etree.HTML(f)
it=p.xpath("//input[@name='lt']/@value")
exe=p.xpath("//input[@name='execution']/@value")
eventId=p.xpath("//input[@name='_eventId']/@value")
jid=p.xpath("//form/@action")
jid=str(jid[0])

#提取postdata#
d={'username':'leiyang_ace@163.com',
	'password':'1005931665asd'}
d['lt']=str(it[0])
d['execution']=str(exe[0])
d['_eventId']=str(eventId[0])
#构造post url
url=url+jid

#进行登陆
gg=s.post(url,data=d,headers=headers)
#重定向
tt=s.get("http://www.csdn.net/")
print(tt.status_code)
#进入个人主页
hh=s.get("http://write.blog.csdn.net/postlist").content.decode('utf-8')
print(hh)



```