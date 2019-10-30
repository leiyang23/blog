---
title: requests模拟登陆京东
description:  
date: 2019-10-30 18:02  
categories:
- 实例   
tags:  
- python  
 
---
requests模拟登陆京东
==
>环境：python 3.6.1 | requests 2.18.4  |  lxml 4.0.0 | pillow 4.2.1
>时间：2017-10-1 可用

-

>以前在一次京东秒杀活动中使用selenium模拟登陆过京东，但最后没抢到，不知是运气不佳还是selenium太慢，这次用requests不知能否成功，以下只是模拟登陆的具体思路和代码。



>**模拟登陆三个要点：目标网址（是否需要构造）、post数据（包含验证信息）、头部信息**

一、思路
>模拟登陆大致模式分三个步骤
>1、试探
>打开浏览器调试工具（我用的是Firefox+chrome），在登陆页面输入账号、密码时故意输错，查看提交了那些数据，如下图
>![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMDAxMjMxMTExMzAx?x-oss-process=image/format,png)
>在红色方框里的就是我们要提交的数据，注意前面的字段，这些我们基本都可以从源文件里提取出来。
>2、准备数据（post参数）
>我们可以把提取出的数据加到请求里，这里我们要注意提交的网址，在网址后面大部分都要加参数的，这时我们就要分析再构造了；如果有验证码的话还要进行验证码图片的提取，这也是个坑，
>![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMDAxMjMyNzE5NzIx?x-oss-process=image/format,png)
>这里我们可以利用调试工具查看验证码图片地址，对我们分析很有帮助的。
>3、尝试登陆，如果没成功就检查前面的数据是否**全**且**正确**。

二、下面上代码

```
python
# _*_coding:utf-8 _*_
__author__ = 'leiyang'
__date__ = '2017-10-1 8:54'

# 模拟登陆京东
import requests
from lxml import etree
import json
import time
from PIL import Image

s = requests.Session()

headers = {
    'Host':'passport.jd.com',
    'X-Requested-With':'XMLHttpRequest',
    'Content-Type':'application/x-www-form-urlencoded; charset=utf-8',
    'Connection':'keep-alive',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36',
}
#先获取登陆页面，从中提取出post参数
page = s.get("https://passport.jd.com/new/login.aspx", headers=headers).text
html = etree.HTML(page)
uuid = html.xpath("//input[@id='uuid']/@value")[0]
eid = html.xpath("//input[@id='eid']/@value")[0]
fp = html.xpath("//input[@id='sessionId']/@value")[0]
_t = html.xpath("//input[@id='token']/@value")[0]
loginType = html.xpath("//input[@id='loginType']/@value")[0]
pubKey = html.xpath("//input[@id='pubKey']/@value")[0]
sa_token = html.xpath("//input[@id='sa_token']/@value")[0]

# 获取验证码图片
img_url = "https://authcode.jd.com/verify/image?a=1&acid=" + str(uuid) + "&uid=" + str(uuid) + "&yys=" + str(
    int(time.time() * 1000))
img_headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100Safari/537.36',
    'Host': 'authcode.jd.com',
    'Accept': '*/*',
    'Cookie': '__jdu=1680020371',
    'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
    'Accept-Encoding': 'gzip, deflate, br',
    'Connection': 'keep-alive',
    'Referer': 'https://passport.jd.com/new/login.aspx?ReturnUrl=https%3A%2F%2Fwww.jd.com'
}
re = s.get(img_url, headers=img_headers )
with open("d://jingdong_img.jpeg", "wb") as f:
    f.write(re.content)
    
# 如果你想让验证码自动弹出来就把下面2行的注释去掉
# im = Image.open("d://jingdong_img.jpeg")
# im.show()

verify_code = input("输入验证码：")

#把上面我们准备的数据填充到字典里
data = {
    'uuid': uuid,
    'eid': eid,
    'fp': fp,
    'loginname':'[填入你的账号名称]',
    '_t': _t,
    'loginType': loginType,
    'nloginpwd': '[你的密码]',
    'pubKey': pubKey,
    'sa_token': sa_token,
    'chkRememberMe': '',
    'authcode': verify_code,
}
post_url = "https://passport.jd.com/uc/loginService?uuid=" + str(
    uuid)+"&r=0.6335209684602039&version=2015"
r = s.post(post_url, headers=headers, data=data)
print(r.text)#如果出现'success'字样，就算成功了。

g=s.get("http://home.jd.com")
print(g.text)#进入个人中心，验证是否成功。

```
>至此模拟登陆京东完毕，总结一下要注意的点

 >1. 验证码地址和登陆提交地址都需要构造
> 2. 获取验证码的头部信息与登陆提交时的头部信息不同
> 3. Firebox调试工具是个好用且方便的利器，熟练使用可以更快的分析

由于需要手动输入验证码，如果要秒杀的话肯定来不及，我的思路是提前5分钟让程序先跑起来，在程序中加定时time.sleep(time),time等于秒杀时间减去当前时间。ok！但愿我有一个好运气，抢到一些什么，方不辜负我这一整天的心血。2017-10-1。