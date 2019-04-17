---
title: "HTTP 请求与响应之请求数据获取"
description: "Go中获取请求的内容的多种方式"
date: 2019-04-16T22:53:16+08:00
draft: true
---

以下的几种方式都是基于vscode中go插件的helloweb来使用
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

#### 获取请求参数

对于请求数据的获取，归根结底就是要将请求到的参数解析到`url.Values`类型中，然后通过该类型的方法来获取数据.
#### 使用`URL.Query`方法

对于一个GET请求`localhost:8080?page=1&per_page=15`,

```go
func greet(w http.ResponseWriter, r *http.Request) {
	v := r.URL.Query()
	fmt.Fprintf(w, "page value:%s per_page value: %s", v.Get("page"), v.Get("per_page"))
}
```

#### 使用`ParseForm`方法
```go
func greet(w http.ResponseWriter, r *http.Request) {
	err := r.ParseForm()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Fprintf(w, "page value:%s per_page value: %s", r.Form.Get("page"), r.Form.Get("per_page"))
}
```
这里先使用`ParseForm`方法解析请求参数，并将解析之后的查询参数赋值给请求的`Form`字段上,而请求的`Form`就是url包的`Values`结构体，因此可以使用该结构的方法来获取参数

其实不仅仅是对于GET方法，对于HTTP请求的`POST,PUT,PATCH`方法，都能够将请求体解析到`url.Values`结构体上。而且优先级高于`GET`请求。

但是需要注意的是如果使用除`GET`方法外的其他方法，需要设置请求头为`Content-Type:application/x-www-form-urlencoded`,否则无法获取到请求数据

#### 使用`PostFormValue`方法
使用该方法可以获取`POST,PUT.PATCH`方法的请求体，忽略请求地址中的查询参数,当然需要将请求头部的`Content-Type`设置成`application/x-www-form-urlencoded`

```go
func greet(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "page value:%s per_page value: %s", r.FormValue("page"), r.FormValue("per_page"))
}
```

### 文件请求
使用`FormFile`方法可以获得上传的问题，依据约定上传文件的时候需要设置`Content-Type`为`multipart/form-data`
```go
func greet(w http.ResponseWriter, r *http.Request) {

	fread, finfo, err := r.FormFile("file")
	if err != nil {
		log.Fatal(err)
	}
	defer fread.Close()
	newFile, err := os.OpenFile("./"+finfo.Filename, os.O_WRONLY|os.O_CREATE, 0666)
	if err != nil {
		fmt.Fprintf(w, "error %s", err)
	}
	defer newFile.Close()
	io.Copy(newFile, fread)
	fmt.Fprintf(w, "file upload success")
}
```
1. 以上先使用`FormFile`方法来获取上传的文件信息,该方法为
```
func (r *Request) FormFile(key string) (multipart.File, *multipart.FileHeader, error)
```
- `multipart.File` 是一个文件接口，该接口实现了底层io包的`Reader, ReaderAt,Seeker,Closer`接口,

- `multipart.FileHeader` 是一个结构体，该结构体包含`Filename(文件名)`，`Header(文件类型)`,`Size(文件大小)`

1. 然后使用`defer`关键字关闭文件

1. 接收以上传的文件名为名称在当前目录下创建一个同名文件，并且该文件为新创建只读形式，并赋予`0666`的权限

1. 使用`defer`关键字关闭新创建的文件

1. 使用`io.Copy`方法，将上传的文件内容复制新建的文件中


#### 获取请求头
在Go中，请求头的类型为以string作为键，以字符串切片作为值的map
```go
type Header map[string][]string
```
与`url.Values`的类型一致
```go
type Values map[string][]string
```
而且也提供了相同的方法来获取和设置其中的参数

- Add
- Del
- Get
- Set