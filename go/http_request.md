---
title: "HTTP 请求与响应之请求过程"
description: "了解请求的过程，对于联合使用之前的请求头，消息体，cookie有很大的帮助"
date: 2019-04-19T23:46:33+08:00
draft: true
---

![request_message](/go/images/http_request.png)

一个HTTP 请求的大致流程如下：

```
      Request                                     Response
Client------>Service[Multiplexer(router=>handler)]--------->Client
```
- 首先客户端发送一个请求到服务器

- 服务器接收到请求以后，使用Multiplexer(多路复用器)解析请求，获取请求的路径，
然后将请求分发到之前注册的与路由相对应的handler(处理句柄)上

- 创建响应，并向客户端发送响应。

下面是一个简单的Go实现的web服务器
```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func greet(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World! %s", time.Now())
}

func main() {
	http.HandleFunc("/", greet)
	http.ListenAndServe(":8080", nil)
}
```

在以上的代码中:

- `greet` 是一个handler函数

- `http.HandleFunc("/", greet)` 是将路由与处理函数进行绑定

- `http.ListenAndServe(":8080", nil)` 时开启一个服务，并且监听`8080`端口

其实`ListenAndServe`函数接收两个参数,该函数原型如下:

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```
由原型可知，该函数接收两个参数一个是监听的地址`addr`，另一个是Handler接口，该接口的原型如下:

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
在这里并没有看到所谓的`Multiplexer(多路复用器)`，为何可以传递`nil`进去呢,其实在其背后是因为使用`DefaultServeMux`,至于如何使用到这个的，
在解释这个之前，需要了解一下多路复用器中路由与处理句柄是如何建立联系的。go中包含两种方式来进行绑定，一个是接口的方式`Handler`,一个是类型的方式

### Handler
由于Handler是一个接口，凡是实现了该接口的类型，都可以当作是该类型，因此只要实现这个方法就可以注册为请求处理句柄,下面试着改写一下之前的服务
```go
type Hello int

func (h Hello) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World! %s", time.Now())
}

func main() {
	var h Hello = 2
	http.Handle("/", h)
	http.ListenAndServe(":8080", nil)
}
```
以上，声明了一个Hello类型，然后使得该类型实现了`ServeHTTP`方法，因此，该函数可以作为请求的处理句柄

### HandleFunc
先看一下这中方法所接收参数的原型：
```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```
在该方法中，使用`DefaultServeMux.HandleFunc`方法将接收路由字符串和处理方法。再进一步观察`DefaultServeMux.HandleFunc`,发现它的原型如下：

```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}
	mux.Handle(pattern, HandlerFunc(handler))
}
```
这个时候，可以观察到使用`HandlerFunc`方法来进行类型的转换，在进一步观察的话，可以看到类型`HandlerFunc`的原型如下：

```go
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}

```
由上可以看到`HandlerFunc`作为一个类型，也是实现了`ServeHTTP`方法，才能与路由进行绑定，这与我们自己声明一个类型，并实现`Handler`接口的效果是一样的。

### DefaultServeMux

在查看`http.ListenAndServe`原型的时候，可以看到其实际上是实例了一个`http.Server`,而在该结构体上包含一个`handler`的字段，其类型是`Handler`,

在包中也说明了如果`http.ListenAndServe`第二个参数为空的话，那么将会使用`http.DefaultServeMux`作为默认的handler。而且在使用`http.HandleFunc`的时候

确实也是使用的是`DefaultServeMux.HandleFunc`方法来将路由和处理句柄进行绑定。

除了默认声明多路复用之外，当然也可以手动进行声明，go提供了简便的方法来进行声明,改写最先的代码

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func greet(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World! %s", time.Now())
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", greet)
	http.ListenAndServe(":8080", mux)
}
```


#### 参考文档

- [Understanding Go Standard Http Libraries : ServeMux, Handler, Handle and HandleFunc](https://rickyanto.com/understanding-go-standard-http-libraries-servemux-handler-handle-and-handlefunc/)

- [Golang构建HTTP服务（一）--- net/http库源码笔记](https://www.jianshu.com/p/be3d9cdc680b)