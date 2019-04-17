---
title: "GoLang-Interface"
description: 凡是实现了接口所定义的方法的类型，都认为是实现了接口
date: 2019-04-02T17:06:50+08:00
draft: true
---

### 接口的定义

```go
type areaAndPerimeter interface {
	area() float64
	perimeter() float64
}
```

如上所示，定义一个接口的通用方式是`type interfaceName interface{}` ,上面的代码定义了一个名为`areaAndPerimeter`的接口，该接口定义了两个方法`area`和`permimeter`来计算面积和周长。



### 接口的实现

要实现一个接口，只需要实现接口所定义的所有方法即可

```go
type quadrangle struct {
	width, height float64
}

func (q quadrangle) area() float64 {
	area := q.width * q.height
	return area
}

func (q quadrangle) perimeter() float64 {
	perimeter := 2 * (q.width * q.height)
	return perimeter
}

type circular struct {
	r float64
}

func (c *circular) area() float64 {
	area := Pi * c.r * c.r
	return area
}

func (c *circular) perimeter() float64 {
	perimeter := 2 * Pi * c.r
	return perimeter
}
```

以上分别定义了两个类型`quadrangle`和`circular` 他们都实现了接口，稍微有一点区别的是`quadrangle`采用的是值接收者，`circular`使用的是指针的接收者,然后我们就可以如下使用

```go

var ap areaAndPerimeter

q := quadrangle{
	width:  3,
	height: 4,
}
fmt.Println(q.area())

ap = q
fmt.Println(ap.area())

c := circular{
	r: 5,
}

fmt.Println(c.area())

ap = &c
fmt.Println(ap.area())
```
由此可见，实现了接口定义的方法也就实现了接口，上面作为指针接收者的类型与值类型接收的两个类型赋值给接口的方式不一样
一个直接赋值，一个使用`&`获取地址后赋值，这是因为对于`circular`类型来说，实现接口的是指针类型而不是值类型。

### 接口的值

对于接口来说，其值有两部分组成`(value, type)` 

- `value` 表示的是实现该接口的具体的类型的值

- `type` 表示的是实现该接口的类型的类型

```go
var ap areaAndPerimeter

q := quadrangle{
	width:  3,
	height: 4,
}
fmt.Printf("the q  type is:%T value is:%v \n", q, q)

ap = q
fmt.Printf("the ap type is:%T value is:%v \n", ap, ap)

c := circular{
	r: 5,
}

fmt.Printf("the c type is:%T value is:%v \n", c, c)

ap = &c
fmt.Printf("the ap type is:%T value is:%v \n", ap, ap)

```
以上将会打印出结果:
```
the q  type is:main.quadrangle value is:{3 4} 
the ap type is:main.quadrangle value is:{3 4} 
the c type is:main.circular value is:{5} 
the ap type is:*main.circular value is:&{5}
```

以上情况都是在底层类型值不为`nil`的情况下进行计算，当底层类型的值为`nil`时，会出现什么情况
```go
var ap areaAndPerimeter

var q quadrangle

fmt.Printf("the q  type is:%T value is:%v \n", q, q)

ap = q
fmt.Printf("the ap type is:%T value is:%v \n", ap, ap)

fmt.Println("area is:", ap.area())
```
以上将会输出
```
the q  type is:main.quadrangle value is:{0 0} 
the ap type is:main.quadrangle value is:{0 0} 
area is: 0
```
以上结果可以看出，如果实现接口的类型的值为`nil`,那么调用方法的时候，将会使用类似于`nil.methodName`的方式进行调用

还有一种情况是，如果接口的值为`nil`的情况下会出现什么问题
```go
func main() {
	var ap areaAndPerimeter

	fmt.Println(ap.area())
}
```
如果执行以上代码，将会出现一个错误
```
panic: runtime error: invalid memory address or nil pointer dereference
```


### 类型断言

在接口中，当我们需要判断一个类型的类型的时候，将会使用`i.(T)`的形式，

- `i` 表示需要判断类型的值

- `T` 断言`i`所代表的类型

```go
	var ap interface{} = quadrangle{
		width: 3,
	}
	fmt.Println(ap.(circular))
```
如上，先声明一个空接口类型的变量`ap`,然后赋值`quadrangle`类型，然后我们定义该类型了为`circular`，运行将会得到如下错误：
```
panic: interface conversion: interface {} is main.quadrangle, not main.circular
```
为了解决这个问题，与判断`map`类型中的元素是否存在一致，引入一个`ok`变量
```go
a, ok := ap.(circular)
if ok {
	fmt.Println("the ap type was circular")
} else {
	fmt.Printf("the a type is: %T, value is:%v", a, a)
}
```
以上将会输出：
```
the a type is: main.circular, value is:{0}
```
可以看到，如果类型判断错误，`ok`的值为false, 返回的值为对应接口类型的零值。
