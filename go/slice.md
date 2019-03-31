---
title: "GoLang-切片"
description: GoLang语言中的切片
date: 2019-03-27T20:10:55+08:00
draft: true
---

​	由于数组的长度属性属于数组类型的一部分，因此限定了数组的灵活性，为了解决这个问题，Golang中引入了切片作为数组的一个补充。切片本身不存储任何的数据，它只是底层数组的一个反射，因此对于切片的修改将直接作用到底层的数组上。

### 切片的属性

对于切片而言，长度，容度是我们特别重要的两个属性

- 长度，指的是切片中元素的数量，可以使用`len`函数获取
- 容度，指的是切片创建的时候从切片起始位置到数组末尾所占元素的总和，可以使用`cap`函数获取

因此，对于一个切片而言，其长度<=容度

### 切片的声明

1.  使用内置的`make`函数直接生成

   ```go
   s := make([]int, 3, 4)
   fmt.Println(s)
   ```

   使用`make`函数创建切片的时候可表达为`make([]T,len,cap)`，

   - `[]T` 用来指明切片底层数组所保存的元素类型。
   - `len` 用来标识切片底层数组的长度。
   - `cap`代表的是切片的容度,通常可省略，省略的时候切片的长度=容度

   实际在创建一个切片的时候会先创建一个底层数组，然后将底层数组反射成切片

1. 基于已存在的数组，直接生成切片

   ```go
   a := [...]int{1, 2, 3, 4, 5, 6}
   s := a[2:4]
   ```

   以上将会输出`[3 4]`, 基于数组创建切片的时候，切片的取值范围通用表达式为`arr[start:end]`

   - `start`的值决定了切片的容度大小

   - `end` 的值的取值范围为`0~len(arr)`

   - 如果需要取最前端或者最末端的时候，可以留空,比如`[:],[start:],[:end]`

   - 当创建切片以后，切片中的元素所以将会重新从0开始编号

     

2. 基于已存在的切片来创建

   ```go
   a := [...]int{1, 2, 3, 4, 5, 6}
   s1 := a[4:4]
   fmt.Printf("s1 len:%d, cap:%d \n", len(s1), cap(s1)) //输出：s1 len:0, cap:2 
   fmt.Println(s1) //输出：[]
   
   s2 := s1[:2]
   fmt.Printf("s2 len:%d, cap:%d \n", len(s2), cap(s2)) //输出s2 len:2, cap:2 
   fmt.Println(s2) //输出 [5 6]
   
   s3 := s1[:4] //这里将会显示一个：slice bounds out of range的错误
   fmt.Printf("s3 len:%d, cap:%d \n", len(s3), cap(s3))
   ```

   当我们基于已存在的切片去创建一个切片的时候，创建的切片还是底层数组的一个反射。但是需要注意的是切片的`end`如果其值超过了父切片的容度，将会出现下标索引取值出现问题。

   

### 切片的操作

由于切片是数组的反射，因此我们对于切片的所有的操作都将直接作用于底层数组，同时也会直接影响到基于改底层数组

的其他切片。例如下面这个例子

```go
a := [...]int{1, 2, 3, 4, 5, 6}
s1 := a[2:4]
s2 := a[1:4]
fmt.Println("before a:", a)
fmt.Println("before s1:", s1)
fmt.Println("before s2:", s2)
s1[0] = 23
fmt.Println("after a:", a)
fmt.Println("after s1:", s1)
fmt.Println("after s2:", s2)
```

以上将会直接输出为：

```
before a: [1 2 3 4 5 6]
before s1: [3 4]
before s2: [2 3 4]
after a: [1 2 23 4 5 6]
after s1: [23 4]
after s2: [2 23 4]
```



当我们对切片进行追加元素的时候，会出现两种情况，元素的长度小于切片的容度，元素的长度大于切片的容度

```go
a := [...]int{1, 2, 3, 4, 5, 6}
s1 := a[:3]
fmt.Printf("s1 len:%d, cap:%d \n", len(s1), cap(s1))
fmt.Println(s1)

s1 = append(s1, 1) 
fmt.Printf("s1 len:%d, cap:%d \n", len(s1), cap(s1))
fmt.Println(s1)

s1 = append(s1, 1, 2, 3, 4, 5)
fmt.Printf("s1 len:%d, cap:%d \n", len(s1), cap(s1))
fmt.Println(s1)
```

以上将会输出

```go
s1 len:3, cap:6 
[1 2 3]
s1 len:4, cap:6 
[1 2 3 1]
s1 len:9, cap:12 
[1 2 3 1 1 2 3 4 5]
```

在Go中，使用`append`方法对切片的末尾添加元素，其通用表达式为：`append(slice,item....Type)` ,可以往切片中一次性添加一个或者多个与切片相同类型的元素, 当然也可以使用`append`将两个切片进行合并，就像`append(s1,s2...)`。

在对切片进行添加元素的时候，如果添后的元素总数量大于切片的容度，将会对元素进行扩容，而对于扩容的规则，可以参考: [golang 切片扩容的探讨](https://studygolang.com/articles/16052?fr=sidebar)和[slice.go文件](https://github.com/golang/go/blob/master/src/runtime/slice.go)



