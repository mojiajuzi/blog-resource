---
title: "GoLang-Method"
description: 方法是多了接收者的函数，接收者可以在函数内部直接访问
date: 2019-03-31T22:35:32+08:00
draft: true
---

函数和方法在参数的传递和值的返回方面是一致的，不同的是方法多了一个接收者

### 方法的声明

```go
// 声明四边形结构体
type quadrangle struct {
	width, height int
}

// 使用结构体作为接收者
func (q quadrangle) area() int {
	area := q.width * q.height
	return area
}

func main() {

	q := quadrangle{
		width:  3,
		height: 4,
	}
    //直接访问方法
	area := q.area()
	fmt.Println(area)
}
```

在以上示例中，

1. 先声明一个`quadrangle`作为结构体，该结构体有两个字段`width`,`height`
2. 声明一个方法`area`,该方法的接受是一个`quadrangle`类型的变量,该方法返回计算后的面积
3. 在方法中，直接访问接收者的字段来计算面积
4. 在`mian`函数中，我们使用`.`语法来访问一个接收者的方法

当然。一个接收者可以包含多个方法,例如，除了计算面积，还可以添加一个计算周长的方法

```go
func (q quadrangle) perimeter() int {
	perimeter := 2 * (q.width * q.height)
	return perimeter
}
```



那么问题来了，在什么时候使用方法，在什么使用函数呢

- 当我们需要在一个类型上实现其他语言的类方法的时候，可以使用方法
- 当我们在同一个包的不同类型上使用相同的名字时，需要使用方法，例如：计算圆的面积和计算四边形的面积方式是不一样的，这个时候我们就可以使用不同的类型作为相同方法名称的接收者

### 接收者

对于接收者而言，其分为两种类型：指针接收者和值接收者，这两者的主要区别在于，如果在方法中修改了接收者的值，是否会影响接收者本身,下面是一个例子

```go
func (q quadrangle) area() int {
	area := q.width * q.height
	q.height++
	return area
}

func (q *quadrangle) perimeter() int {
	perimeter := 2 * (q.width * q.height)
	q.width++
	return perimeter
}
```

以上稍微修改一下两个方法，面积使用值作为接收者，周长的计算则使用指针类型的接收者，在这两个方法中，都对类型的字段进行修改，然后再`main`函数中分别进行调用

```go
func main() {

	q := quadrangle{
		width:  3,
		height: 4,
	}
	fmt.Println("before q:", q)
	fmt.Println(q.perimeter())
	fmt.Println("after q:", q)

	fmt.Println("before q:", q)
	fmt.Println(q.area())
	fmt.Println("after q:", q)
}
```

以上的输出结果为：

```
before q: {3 4}
24
after q: {4 4}
before q: {4 4}
16
after q: {4 4}
```

由此可以看到，值类型的接收者，如果在方法中改变了其字段的值，并不会作用与方法之外的接收者本身，也就是说它仅仅是一个值的拷贝

### 内嵌结构体方法

对于内嵌结构体来说	，如果内嵌的结构体有方法，那么可以方法属性一样，使用`.`语法直接访问该方法，当然也可以提权

```go
func main() {
	press := Press{
		name:   "电子工业出版社",
		city:   "北京市",
		detail: "万寿路南口金家村288号华信大厦",
	}
	book := Book{
		name:   "编码",
		author: "Charles Petzold",
	}
	book.Press = press

	detail := book.fullAddress()
	fmt.Println(detail)
}

type Press struct {
	name   string
	city   string
	detail string
}

func (press Press) fullAddress() string {
	fulladdress := press.city + press.detail
	return fulladdress
}

type Book struct {
	name   string
	author string
	price  int
	Press  //匿名字段
}
```

以上`Press`结构体有有一个获取完成地址的方法`fullAddress`, `Press`结构体又内嵌于`Book`结构体之内，最后通过`book.fullAddress()` 的方式来直接获取内嵌结构体中字段的方法。

Note: 对于非本地类型(也就是不在包中定义的类型)，无法作为函数的接收者,比如:int,string类型等

```go
func (a string) add(b string) string {
	c := a + b
	return c
}
```

