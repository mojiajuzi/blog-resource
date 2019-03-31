---
title: "GoLang-Map"
date: 2019-03-28T22:47:12+08:00
description: Golang中的Map
draft: true
---

`map`是Go语言的一种内建类型，该类型将一个键和值相互关联起来，通过该键获取其所存储的值。因此我们首先遇到的问题是，哪些类型能够作为key来进行使用,总的来说，所有可以直接使用比较操作符(比如: `==,!=,<,<=,>,>=`)进行比较的类型，都可以用来作为key来使用。

- 布尔值（bool）
- 整型（integer）
- 浮点型(float)
- 复数
- 字符串
- 指针
- channel (通道)值
- interface(接口)
- struct(结构体)
- array(数组)

### map的声明

对于一个map来说，其元素由key和value组成，因此一个map的声明如下

```go
var m map[string]string
```

其通用表达式为`map[Key_Type]value_type` ,map的零值为`nil` ,为了验证这个问题，可以做如下的实验

```go
    var m map[string]string
    if m == nil {
        fmt.Println(m)
    } else {
        fmt.Println("b")
    }
```


以上结果将会直接输出`map[]`，但是我们无法在其中添加元素，因为map的零值为`nil`,为了解决这个问题，可以有一下两种方式

1. 使用`make`函数来进行声明

    ```go
       var m = make(map[string]string)
       if m == nil {
           fmt.Println("a")
       } else {
           fmt.Println(m) //将会输出:map[]
       }
    ```

2.  声明的时候，直接进行赋值

    ```go
        var m = map[string]string{
           "a": "a",
           "b": "b",
        }
        if m == nil {
           fmt.Println("a")
        } else {
           fmt.Println(m) //输出:map[a:a b:b]
        }
    ```
	声明的同时进行赋值的话，需要注意的是，其中的最后一个元素必须以`,`进行结尾，否者将会出现错误

### map的操作

对于一个map而言，我们常用的操作就是添加元素，删除元素，查找元素，遍历。

1. 添加元素，与切片一样，直接操作key值对map中的元素进行操作

    ```go
    var m = make(map[string]string)
    m["hello"] = "world"
    fmt.Println(m)
    m["hello"] = "中国"
    fmt.Println(m)
    ```
    以上将会直接输出：

    ```go
    map[hello:world]
    map[hello:中国]
    ```
    因此，如果key不存在则会直接添加，存在的话将会覆盖原有的值

    

1. 删除元素,对于`map`中的元素，使用delete来进行元素的删除

       ```go
       delete(map, key)
       ```
     如果元素存在将会从map中删除，如果元素不存在，与元素存在时的删除效果一致  

1. 查找元素,由于查找的时候，如果元素不存在，那么将会返回0,这里将会存在一个问题，那就是如果值为0的时候，我们将无法判断元素是否存在，因此，返回值我们需要多加一个参数来反应元素是否存在。

    ```go
    var m = make(map[string]string)
    
    value, ok := m["hello"]
    if ok == true {
        fmt.Println(value)
    } else {
        fmt.Println("value not exists")
    }
    ```

    

1. 遍历,与数组和切片一样，可以使用`for...range`方式来遍历元素

    ```go
    var m = make(map[string]string)
    
    m["a"] = "a"
    m["b"] = "b"
    m["c"] = "c"
    for key, value := range m {
        fmt.Printf("the key:%s, value:%s \n", key, value)
    }
    ```
    与数组和切片的有序不同的是，map中的元素是无序的，因此每一次打印元素的顺序可能会略有不同

    

    除了一下基本的操作，map还有一些其他的特性:

    - 和数组，切片一样，map中的元素数量，可以使用`len`函数直接获取

    - 与切片一样，map也是一个反射，对于相同底层的map来说，改变其中某个map的值，其他的map中的值也会相对应的改变。
    - map是不能使用`==`操作符号来进行直接比较的，如果要对两个map进行比较，其中的一种方式是循环对其中的元素逐个的进行比对。