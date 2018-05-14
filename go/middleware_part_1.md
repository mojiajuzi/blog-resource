---
title: "Golang Middleware Part 1"
date: 2018-02-06T23:30:40+08:00
draft: true
tags: ["Middleware", "Go"]
---

## 如何在Golang中实现中间件-Part 1

当使用`net/http`包实现服务的时候，一般使用的是如下的两中处理方式:

- http.HandleFunc
- http.Handle

### http.HandleFunc

#### 分析

当使用这种方式的时候，其接受两个参数，一个是字符串格式的匹配符(pattern),另外一个就是`func(ResponWrite, *Request)`,
因此只要我们的中间件中返回该类型，那么中间件就是可以实现的

{{< highlight golang >}}
func main(){
    http.HandleFunc("/", Hello)
	http.ListenAndServe(":8080", nil)
}

func Hello(w http.ResponseWriter, r *http.Request)  {
	fmt.Print("hello")
}
{{< /highlight>}}
当我们运行如上的程序的时候，就会打印出`hello`这个结果，说明我们的写法是没有问题的

#### 实现

接下来，我们需要定义我们的中间件，它需要接收一个｀ http.HandlerFunc｀类型，并且返回一个` http.HandlerFunc`这样才能被使用

{{< highlight golang >}}
func MyMiddleware(next http.HandlerFunc)http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("middleware")
        //doSomethinds
		next.ServeHTTP(w,r)
	}
}
{{< /highlight >}}
由于调用`next.ServeHTTP(w, r)`等效于调用`next(w, r)`,处理完后会返回响应`w`,最终相应会传递到最外层的匿名函数，从而最终会返回到客户端


### http.Handle

#### 分析
`http.Handle`接受两个参数，一个是匹配符，另外一个是`http.Handler`，当我们查看源码的时候，发现其是一个接口类型

{{< highlight golang >}}
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
{{< /highlight >}}
因此我们最终得返回一个实现了该接口的类型，假设我们首先定义一个新的结构体,并为其添加一个｀ServiceHTTP｀

#### 解决

{{< highlight golang >}}
type MyResponse struct {
	next http.Handle
	Code int64
	Msg string
	Errors []string
	Data map[string]interface{}
}

func (res *MyResponse)ServeHTTP(w http.ResponseWriter, r *http.Request)  {

	result, err := json.Marshal(res)
	if err != nil {
		fmt.Println(err.Error())
	}
	w.Write(result)
}
{{< /highlight >}}

紧接着我们定义两个处理函数

{{< highlight golang >}}
func Hello(w http.ResponseWriter, r *http.Request)  {
	fmt.Println("hello")
}

func World(w http.ResponseWriter, r *http.Request)  {
	fmt.Println("world")
}
{{< /highlight >}}

然后在主函数里面进行调用

{{< highlight golang >}}
    var res = new(MyResponse)
    res.next = Hello
    http.Handle("/hello", res)
    res.next = World
	http.Handle("/world", res)
	http.ListenAndServe(":8080", nil)
{{< /highlight >}}
然后你会发现诡异的事情发生了，无论你访问哪一个路由地址，最终打印的都只是`world`,这是因为Golang是一门静态的预编译语言，编译完成后，｀res｀总的next属性
将会永远指向`World`的地址,

因此我们可以做一个映射，将next变成map类型，但是想一想，如果路由非常多的话，那将是一件多可怕的事情，因此我们需要另辟蹊径，有没有其他的办法可以实现,查看源码我们可以发现
之前我们所使用过的`http.HandleFunc`与之相对应的还有一个类型`http.HandlerFunc`，该类型实现了`ServerHTTP`方式

{{< highlight golang >}}
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
{{< /highlight >}}
接下来我们利用Goalng中的强制类型转换，就可以写出如下的中间件

{{< highlight golang >}}
func MySencondMiddleware(next http.HandlerFunc) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		//do somethings
		next.ServeHTTP(w, r)
	})
}
{{< /highlight >}}