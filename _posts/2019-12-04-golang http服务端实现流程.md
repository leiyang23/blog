---
title: golang http服务端实现流程   
description:  
date: 2019-12-04 0:31  
categories:  
- golang

tags:  
- net/http
 
---
    	
### net/http 包实现服务端流程
核心就是 实现一个路由器，可以把我们写的处理单元和路由建立映射。     
在 net/http 中， ServeMux 就是我们要找的路由器。ServeMux 是一种数据结构，其中一个叫 m 的 map 类型就用来存储
处理单元和路由的映射，ServeMux 又关联一系列的方法，主要的有一下几个：    
- handle   
将 URL 和 Handler 建立映射。用来注册我们的处理逻辑。
- ServeHTTP   
客户端访问监听的端口后就会调用 ServeMux 的 ServeHTTP 方法。是 路由器 的触发函数。

- Handler 和 handler   
主要用来获取请求的 路径

- match    
根据 路径 来找到注册的 Handler。  
-----
再来说下 Handler，这是一个 接口，包含一个 ServeHTTP 方法。也就是说 只要实现了 ServeHTTP 这个方法，就是 ServeMux
“合规”的处理单元。 但是我们习惯于写 函数 来处理一个请求，于是官方定义了一个函数类型的数据结构 `http.HandlerFunc`，这种结构
实现了 ServeHTTP 方法，同时在 ServeHTTP 中也是调用的 函数本身。我们可以**将我们写的函数强制转换程这种数据结构**，
 这样就实现了 Handler，真是奇妙的实现！


```golang
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {

	fmt.Println("开始了，哈哈")
	mux := http.ServeMux{}	
	mux.Handle("/hi", http.HandlerFunc(hi))
	http.ListenAndServe(":8000", &mux)
}


func hi(w http.ResponseWriter, r *http.Request) {
	log.Println("HI")
}

```   
![http 调用流程图](http://qiniu2.freaks.group/golang%20net_http%20server.jpg)

> 雪碧，提神醒脑，肝代码必备。