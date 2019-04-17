---
title: "HTTP 请求与响应之URL"
description: "net/url包的说明与使用"
date: 2019-04-14T22:46:11+08:00
draft: true
---

在web中，一个常见的地址格式为
```
[scheme:][//[userinfo@]host][/]path[?query][#fragment]
```
- scheme: 协议，通常为`http`或者`https`

- userinfo 请求的用户名以及用户密码信息

- host: 主机地址，通常使用域名替代`baidu.com`或者直接使用ip地址`192.168.1.1`

- path: 请求的地址路径，使用`/`进行分割,比如`/user/login`

- query: 查询字符，以`?`开头，通常以k/v键值对的形式出现,例如：`page=1&pre_page=15`，多个查询通常以`&`符号进行分割

- fragment: 锚点，通常用来定位到页面的某个点

或者是
```
scheme:opaque[?query][#fragment]
```
- opaque:表示的是不透明的数据

因此，在go中一个url使用一个结构体来表示
```go
type URL struct {
    Scheme string
    Opaque string
    User *Userinfo
    Host string
    Path string
    RawPath string
    ForceQuery bool
    RawQuery string
    Fragment string
}
```
其中有几个属性是需要特别注意的，那就是`RawPath`,`RawQuery`这两个字段,他们分别表示原始个地址数据以及原始的查询数据
之所以这两者存在是因为url中的path是以`/`进行分割，如果地址或者查询条件使用到这个`/`那么将会对解析造成问题。
因此Rawpath是编码之后的Path,`/`将会编码成`%2F`

#### 常用操作
1. 对于一个地址来说，首先要做的应该对地址的解析，因此一下几个方法从不同的维度对地址进行解析

##### 解析
- Parse
- ParseRequestURI
这两者的区别是，ParseRequestURI只能解析绝对路径的url,如果包含将会忽略锚点，而将锚点当作查询的部分

```go
    func main() {
        u, err := url.ParseRequestURI("https://www.baidu.com/search?q=dotnet#frame")
        if err != nil {
            log.Fatal(err)
        }
        fmt.Println(u.RawQuery)
        fmt.Println(u.Fragment)
    }
```


解析之后，就可以对获得的URL进行操作:

- EscapedPath 获取转义之后的URL.Path
- Hostname 获取除端口号之外的主机名
- IsAbs判断地址是否是一个绝对的地址
- MarshalBinary 将地址转换成[]byte格式
- Parse 将一个url作为接收者来解析地址
- Port 获取请求的端口号
- RequestURI 获取请求的路径以及查询参数

##### 查询参数
在发送http请求或者接收http请求的时候，最常用的一个操作应该是获取请求的参数以及构造请求的参数,在Go中，使用`Values`这个自定义类型来表示，它是一个以字符串作为key，以字符串切片作为value的一个map，定义如下
```go
type Values map[string][]string
```
该结构体提供了以下几个方法来对查询参数进行处理
- Add
- Del
- Encode
- Get
- Set

```go
v := url.Values{}
v.Add("page", strconv.Itoa(1))
v.Set("per_page", strconv.Itoa(15))

fmt.Println("get page after set: value is:", v.Get("page"))

v.Del("page")
fmt.Println("get page after del: value is:", v.Get("page"))

v.Set("q", "google/search")
fmt.Println("encode values", v.Encode())
```
以上运行结果为：
```go
get page after set: value is: 1
get page after del: value is: 
encode values per_page=15&q=google%2Fsearch
```
在这里需要注意的是，尽管`Add`和`Set`都是添加数据，但是`Add`只会往Values里添加新元素，即使已经存在相同的key，也不会替换，`Set`方法将会执行替换的操作

```go
v := url.Values{}
v.Set("page", strconv.Itoa(1))
v.Set("per_page", strconv.Itoa(15))

v.Add("page", strconv.Itoa(2))
v.Set("per_page", strconv.Itoa(10))
fmt.Println("encode values", v.Encode())
```
以上运行结果将会是
```
encode values page=1&page=2&per_page=10
```

除了构建，还需要知道如何解析
```go
u, err := url.Parse("http://example.com?page=1&per_page=10")
if err != nil {
    log.Fatal(err)
}

query, err := url.ParseQuery(u.RawQuery)
if err != nil {
    log.Fatal(err)
}

page, ok := query["page"]
if !ok {
    fmt.Println("page not exists")
}
fmt.Println("page value is:", page)
```

通过配合使用`Parse, ParseQuery`两个函数，将请求参数解析成`Values`，然后就可以通过使用`Values`的方法来获取对应的参数，当然依据`Values`的定义，也可以使用下标的方式获取。