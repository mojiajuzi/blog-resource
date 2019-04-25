---
title: "HTTP 请求与响应之cookie的设置与获取"
description: " 
 HTTP Cookie（也叫Web Cookie或浏览器Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。"
date: 2019-04-18T00:33:34+08:00
draft: true
---

### 简介

通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie使基于无状态的HTTP协议记录稳定的状态信息成为了可能

> - 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
> - 个性化设置（如用户自定义设置、主题等）
> - 浏览器行为跟踪（如跟踪分析用户行为等）

对于服务器而言，当响应一个请求的时候会通过头部设置一个`Set-Cookie`作为key,以cookie的`名称=值`的形式作为value来设置cookie
对于客户端而言，当发送一个请求的时候，会在头部设置以`Cookie`作为key,以分号`;`分隔的多个`名称=值`的形式，
将该路径下的cookie全部发送给服务器。

对于cookie而言，生命周期是其一个重要的属性，按照这个属性，可以将cookie大致分为两类：

- 会话期cookie，关闭浏览器则自动被删除。
- 持久性cookie,在指定的有效期一直有效，直到其过期。

在go的http包中cookie是一个结构体

```go
type cookie struct {
    Name string  //名称
    Value string  //cookie值
    Path string  //作用域
    Domain string 
    Expires time.Time
    RawExpires string
    MaxAge int
    Secure bool
    HttpOnly bool
    SameSite SameSite
    Raw string
    Unparsed []string
}
```
对于其中的字段而言，其说明如下：

- `Name`， `Value`分别为cookie的名称和值

- `Path(路径)`,`Domain(域名)`，两者规定了cookie的作用域。
默认情况下只有当前域名有效，如果设置Domain为`example.com`那么，该域名下的的所有子域名都可以访问到该cookie
默认情况下全站都可以访问，如果指定了具体路径，比如：`/admin`, 那么就只有在admin以及该路径下的子路径才能有效
  

- `Expires(过期时间)`,`MaxAge(有效期)`，两者设置了cookie的声明周期，其中MaxAge的优先级高于Expires,
当MaxAge=0时,表示删除
MaxAge<0时，表示关闭浏览器的时候，将会清除这个cookie
MaxAge>0时，cookie并不随着浏览器的关闭而删除，而是在到达设定的时间时自动删除

- `Secure`,`HttpOnly`,`SameSite`，三者与Cookie的安全相关。 
Secure：规定了cookie是否只能使用https协议进行传输。 
HttpOnly：是否禁止客户端脚本调用cookie
SameSite：防止跨站请求时发送cookie


### 使用

#### 设置cookie

```go
func greet(w http.ResponseWriter, r *http.Request) {
	c := http.Cookie{Name: "hello", Value: "world"}
	http.SetCookie(w, &c)
	fmt.Fprintf(w, "set cookie success,cookie name:%s, value:%s", c.Name, c.Value)
}
```
使用`http.Cookie`创建一个实例，然后使用`http.SetCookie`方法将cookie写入到响应中

#### 获取cookie

```go
func getCookie(w http.ResponseWriter, r *http.Request) {

	c1, err := r.Cookie("hello")
	if err != nil {
		fmt.Fprintf(w, err.Error())
		return
	}
	fmt.Fprintf(w, "get cookie success,cookie name:%s, value:%s", c1.Name, c1.Value)
}
```
使用`r.cookie`可以直接通过cookie的名称获取对应的cookie结构实例,当然还可以通过使用`r.Cookies`方法，将全部的cookie解析到一个切片中

#### 删除cookie

```go
func delCookie(w http.ResponseWriter, r *http.Request) {

	c1, err := r.Cookie("hello")
	if err != nil {
		fmt.Fprintf(w, "get cookie fail")
	}
	c1.MaxAge = -1
	http.SetCookie(w, c1)
	fmt.Fprintf(w, "del cookie success,cookie name:%s, value:%s", c1.Name, c1.Value)
}
```
cookie不能直接删除，只能通过设置其过期时间Expires和MaxAge两个属性来使其过期。
