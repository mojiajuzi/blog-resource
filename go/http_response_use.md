---
title: "在go程序中使用请求和响应"
date: 2019-04-21T17:58:21+08:00
draft: true
---

在程序中经常需要请求其他的API接口来获取数据或者授权，因此在go中发送和接收请求是重要的一个环节。

### 发送请求

#### 使用`http.Client`

##### 使用client.GET
```go
func greet(w http.ResponseWriter, r *http.Request) {

	client := new(http.Client)
	uri := "http://www.example.com"

	// 设置请求参数
	v := url.Values{}
	v.Add("q", "golang")
	query := v.Encode()
	url := uri + "?" + query

	resp, err := client.Get(url)

	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
	}
	defer resp.Body.Close()

	fmt.Fprintf(w, "request uyrl is:%s", url)

}
```

1. 先实例化一个client结构体

1. 然后使用`url.Values`类型添加请求参数，并编码成字符串

1. 然后使用`client.GET`方法发送一个请求

1. 定义了一个defer结构来关闭响应的消息体

Note: 需要注意的是，当使用`client.GET`方法发送的时候是无法设置请求头的。


##### 使用client.Post
```go
client := new(http.Client)
uri := "http://www.example.com"

// 设置请求参数
v := url.Values{}
v.Add("page", strconv.Itoa(1))
v.Add("per_page", strconv.Itoa(10))
body := strings.NewReader(v.Encode())

resp, err := client.Post(uri, "application/x-www-form-urlencoded", body)

if err != nil {
	http.Error(w, err.Error(), http.StatusBadRequest)
	return
}
defer resp.Body.Close()

fmt.Fprintf(w, "request send success")
```

使用`client.Post`发送请求的时候，需要设置的请求的`Content-Type`,而且请求的消息体需要是一个`io.Reader`类型,

与`client.GET`一样，不能够设置请求头

##### 使用`client.PostForm`

```go
client := new(http.Client)
uri := "http://www.example.com"

// 设置请求参数
v := url.Values{}
v.Add("page", strconv.Itoa(1))
v.Add("per_page", strconv.Itoa(10))

resp, err := client.PostForm(uri, v)
if err != nil {
	http.Error(w, err.Error(), http.StatusBadRequest)
	return
}
defer resp.Body.Close()

fmt.Fprintf(w, "request send success")
```
与`client.Post`效果一样，`client.PostForm`其实方法内部是调用了`client.Post`方法，并且对`url.Values`类型进行了编码，其原型为：

```go
func (c *Client) PostForm(url string, data url.Values) (resp *Response, err error) {
	return c.Post(url, "application/x-www-form-urlencoded", strings.NewReader(data.Encode()))
}
```

##### 使用`client.Do`
由于`GET,Post,PostForm`都无法设置请求头，为了满足在特定的情况下设定特殊请求头的需求，因此就需要使用`Do`方法,`client.Do`方法的原型如下:

```go
func (c *Client) Do(req *Request) (*Response, error) {
	return c.do(req)
}
```
可以看到该方法其实接收的就是一个`http.Request`类型的指针，因此只要理解了`http.Request`，就能解决发送请求的问题

###### `http.Request`

```go
type Request struct {
    Method string  //请求的方法

    URL *url.URL // 请求的地址信息

    Proto string //请求的协议,在使用client请求的时候，该参数将会背忽略
    ProtoMajor int
    ProtoMinor int 

    Header Header // 请求头，在使用client请求的时候，部分头部会自动设置，比如` Content-Length`

    Body id.ReadCloser //消息
	
    //获取请求的消息体，常用于当请求发生重定向的时候用来获取之前请求的参数
    GetBody func() (io.ReadCloser, error) 

    // 此次发送内容的长度
    ContentLength int64 

    TransferEncoding []string //编码转换

    Close bool //关闭该请求的连接

    Host string 域名

    Form url.Values //发送表单请求的时候，设置字段

    PostForm url.Values //发送表单请求的时候，设置字段

    MultipartForm *multipart.Form //使用client发送请求的时候，将会忽略该字段，而是使用Body替代

    Trailer Header //指定在请求主体之后发送的附加头

    RemoteAddr string // 该次发起请求的地址，使用client发送的话将会进行忽略

    RequestURI string //请求的uri

    TLS *tls.ConnectionState //TLS请求信息

    Cancel <-chan struct{} //

    Response *Response // 响应
}
```

因此发送一个请求可以有如下的方式:
```go
v := url.Values{}

v.Add("page", strconv.Itoa(2))

body := strings.NewReader(v.Encode())

req, err := http.NewRequest("method", "http://www.example.com", body)
if err != nil {
	http.Error(w, err.Error(), http.StatusNotFound)
}

req.Header.Add("hello", "world")

client := http.Client{}
rep, err := client.Do(req)
if err != nil {
	http.Error(w, err.Error(), http.StatusNotFound)
}
defer rep.Body.Close()

w.Write([]byte("请求成功"))
```

### 接收响应
之前已经提及在go中如何响应请求，其实在go中，响应是一个结构体,目前的主要参数

```go
type Response struct {
	Status string //状态说明

	StatusCode int //状态码，如：200

	Proto string  //协议：如：HTTP/1.0

	Header Header //响应头部

	Body io.ReadCloser //消息体

	ContentLength int64 //内容长度

	Close bool //是否关闭

	Request *Request //获取该响应的请求，其消息体已经被关闭
}
```

Note: 需要注意的是，在获取到请求以后，需要关闭请求体
```go
defer rep.Body.Close()
```


下面是对上面使用`client.Do`处理示例：

```go
var content []byte
len, err := rep.Body.Read(content)

if err != nil {
	//TODO错误处理
	return
}

fmt.Fprintf(w, "response content is:%d", len)
```

或者是使用如下的方式,将获取到的响应直接返回

```go
byteValue, err := ioutil.ReadAll(rep.Body)

if err != nil {
	//TODO错误处理
	return
}
w.Write(byteValue)
```