---
title: golang http服务端实现流程   
description:  
date: 2019-12-04 0:31  
categories:  
- golang

tags:  
- net/http
 
---

// 初始化一个 ServeMux 类型变量，这样我们新建的 mux 变量就具备了很多能力（方法），
	// 我们可以在初始化时提供值，但是 ServeMux 提供了一个方法可以动态地添加。
	// Handle 方法是一个入口方法，用来将 url 和 我们的 Handler 写到 mux 的 map 中。
	// 所谓 Handler 就是一个接口，这个接口只有一个 ServeHTTP 方法，实现了这个方法就行。
		// 监听 8000 端口，每次接收到请求后都会调用 mux 的 ServeHTTP 方法。
    	// 在 ServeMux 的 ServeHTTP 中，会调用 自己的 Handler 方法，解析出 host 和 url，
    	// 然后传递给自己的 handler 方法，handler 方法将 url path 传递给自己的 match 方法，
    	// 而 match 则根据 handle 注册的 url 和 Handler 的映射关系找到对应的 Handler,
    	// 此时回到 mux 的 ServeHTTP 中继续执行，这是继续调用找到的 Handler 的 ServeHTTP 方法，
    
    	// Handler 的 ServeHTTP 方法和 ServeMux 的作用不同
    	// 通过 http.HandlerFunc 将我们写的处理函数强制转换成 http.HandlerFunc 结构。使我们的
    	// 函数一下就实现了 Handler 接口 -- 具备 ServeHTTP 方法。
    	
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