---
title: "GoLang-结构体"
description: 结构体是用户自定义的字段集合，一个结构体是一种自定义类型
date: 2019-03-31T12:45:57+08:00
draft: true

---

### 结构体的声明

对于结构而言，其声明方式为`type StructName struct{}`

```go
type Book struct {
	name   string
	author string
	price  int
}
```

如上所示，我们创建了一个名字叫做`Book`的结构体，他包含三个自定义的字段，在声明字段的时候需要指明字段的类型以及名称，对于相同类型的字段而言，可以放在同一行，比如上面的`name`和`author`字段

```go
name, author string
```

字段的类型，除了常见的`int,string`等类型外，还可以是`pointer,struct,slice`等类型

当然，除了直接定义一个结构体类型，还可以使用匿名结构体，顾名思义，这种结构体，没有名称

```go
var book = struct {
    name, author string
    price        int
}{
    name:   "Dart Time",
    author: "刘未鹏",
}

fmt.Println(book) //将会输出：{Dart Time 刘未鹏 0}
```

对于结构体中的字段而言，也可以省略其名字,在访问的时候，使用其字段类型的名称作为字段的名称来访问,不过这种方式并不推荐使用。




### 初始化和访问

当我们声明结构体之后，就可以将它作为一个类型使用

```go
func main() {
	var book Book
    fmt.Println(book) //将会输出:{  0}

	book.name = "Dark Time"
	book.author = "刘未鹏"

    fmt.Println(book) //输出:{Dark Time 刘未鹏 0}
    
	p := &book

	fmt.Println((*p).author) //刘未鹏
	fmt.Println(p.name) //Dark Time
}
```

以上我们先声明一个类型`Book`的变量`book`

- 未初始化的结构体类型中的值为其字段相对应类型的零值
- 结构体中的字段访问使用的是`.`语法
- 对于指针而言，可以省略`*`来直接访问

### 内嵌结构体

内嵌结构体就是一个结构体作为另外一个结构体的字段

```go
type Press struct {
	name string
	city string
}

type Book struct {
	name   string
	author string
	price  int
	press  Press //出版社结构体作为书结构体的一个字段
}
```

在以上代码中，首先声明两个结构体`Press`和`Book`,然后将press作为Book的一个字段，这个时候可以说，Press内嵌于Book

```go
func main() {

	press := Press{
		name: "电子工业出版社",
		city: "北京",
	}
	book := Book{
		name:   "编码",
		author: "Charles Petzold",
		press:  press,
	}

    fmt.Println(book.name) //输出:编码
    fmt.Println(book.press.name)//输出:电子工业出版社
}
```

对于内嵌于结构体中的结构体的字段访问依旧是通过`.`语法来进行访问的。对于内嵌结构体而言，可以通过提权来简化访问其字段的方式，要实现这种方式，在声明的时候，需要做一下特殊的处理。那就需要使用匿名字段

```go
type Press struct {
	name string
	city string
}

type Book struct {
	name   string
	author string
	price  int
	Press //匿名字段
}
```

那么对于内嵌结构体的访问就可以变成

```go
	press := Press{
		name: "电子工业出版社",
		city: "北京",
	}
	book := Book{
		name:   "编码",
		author: "Charles Petzold",
	}
	book.Press = press

	fmt.Println(book.name)
	fmt.Println(book.city)
	fmt.Println(book.Press.name)

```

注意上面的代码，我们通过`book.city`来访问，原本就属于`book.Press`中的`city`字段，仿佛city字段就是book本身的字段一样，当然，如果book中的字段与内嵌的结构体字段名称相同，那么优先访问的依然是其本身的字段。

结构的比较需要注意一下几点:

- 两个结构体相等，表示其内部相同字段的值相同
- 如果结构体包含不可比较的字段，那么该结构体是不能进行比较的



