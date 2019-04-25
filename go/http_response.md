---
title: "HTTP 请求与响应之响应"
description: "对于http响应来说，除了有和http请求具有相同发的头部，消息体，cookie之外，还有其他一些值得关注的地方"
date: 2019-04-20T21:56:46+08:00
draft: true
---

![request_message](/go/images/http_response.png)

![request_message](/go/images/http_message.png)

一个响应可以由一下几个部分,协议，状态码，状态消息，响应头，消息体组成。

在go中的`http.ResponseWriter`是一个接口,它的原型如下：

```go
type ResponseWriter interface {
    Header() Header

    Write([]byte) (int, error)

    WriteHeader(statusCode int)
}
```

- `Header()` 方法主要用来设置请求的头部信息

- `Write()` 方法主要用来设置响应的消息体

- `WriteHeader`主要用来设置响应的状态码，值范围1xx-5xx

### 状态以及状态码

```go
func greet(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(200)
}
```
这里只需要传递对应的数值就可以了，对应的状态说明包会自动添加

### 请求头
请求头是一个`type Header map[string][]string`类型，该类型实现了和之前说的请求头的获取中的方法因此
```go
func greet(w http.ResponseWriter, r *http.Request) {
	w.Header().Add("Serve", "Go web server")
}
```

### 消息体
消息体是一个`[]byte`类型，因此在发送消息体之前需要将消息体转换成该格式,

对于消息体而言其主要有以下几种格式：string，JSON,XML,FILE

#### 字符串
```go
func greet(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "response string type")
	w.Write([]byte("hello Golang"))
}
```

#### JSON
```go
func greet(w http.ResponseWriter, r *http.Request) {
	var s = make(map[string]string)
	s["a"] = "a"
	s["b"] = "b"

	res, err := json.Marshal(s)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
    }
    w.Header().Set("Content-Type", "application/json")
	w.Write(res)
}
```
需要注意的是:

- 这里遇到错误的话，需要`return`返回，不然的话，程序将会继续执行下面的语句，导致错误的发生
- 对于`w.write`而言，可以多次使用，添加多条响应,需要注意的是，消息体应该是最后添加的，如果在`w.write`之前添加，会不起作用。

- 使用`http.Error`响应程序的错误

#### File
使用`http.ServeFile`方法可以直接读取并返回文件的内容
```go
func greet(w http.ResponseWriter, r *http.Request) {
	filePath := path.Join(".", "index.text")
	http.ServeFile(w, r, filePath)
}
```

单然设置特定的头部信息，可以作为一个静态文件来下载
```go
func greet(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Disposition", "attachment; filename=index.text")

	filePath := path.Join(".", "index.text")
	http.ServeFile(w, r, filePath)
}
```
    
#### 参考文档

- [Golang Response Snippets: JSON, XML and more](https://www.alexedwards.net/blog/golang-response-snippets)

- [HTTP OVERVIEW](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview)