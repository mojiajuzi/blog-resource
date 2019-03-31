---
title: "GoLang-指针"
description: 指针存储的是内存中其他值的内存地址
date: 2019-03-29T23:30:52+08:00
draft: true
---



### 指针声明

在GoLang中声明指针的时候，需要表明指针所存储的内存地址对应值的类型，语法为:`*T`

```go
var p *int
```

在声明指针以后，可以使用`&`操作符来获取一个变量的内存地址

```go
var p *int
fmt.Println("zero value of p is ", p) //将会输出：zero value of p is  <nil>
a := 10
p = &a

fmt.Printf("type of p %T \n", p) //type of p *int 

fmt.Println("address is", p) //address is 0xc000058090

fmt.Println("the pointer point value is:", *p) //the pointer point value is: 10
```

- 通过以上代码可以看到，对于指针来说，其零值为`nil`,

- 使用`&`操作符可以直接获取一个变量的地址
- 通过使用`*`操作符可以直接获取指针所指向的值。



当然也可以直接通过指针来修改其所指向地址所代表的值

```go
var p *int

a := 10

p = &a

fmt.Println(a) //输出 10

*p++

fmt.Println(a) //输出 11

p++ //invalid operation: p++ (non-numeric type *int)
```

这里需要注意的是，在Go中不能够对指针执行算术运算