---
title: "GoLang-Function"
description: 方法是解决特定问题的语句集合
date: 2019-03-31T22:16:11+08:00
draft: true
---

对于一个函数而言，其通常由三部分组成：函数名，参数，返回参数，对于某些函数，也可以没有返回参数

### 函数声明

函数的声明主要使用关键字`func`开头，后面更上函数的参数以及参数的类型，如果由返回参数的话，还需要声明返回参数的类型,比如

```go
func area(width int, height int) int {
	area := width * height
	return area
}
```

以上声明一个计算面积的函数，该函数的名称为`area`, 它包含两个int类型的惨呼，以及它的返回参数是int类型

#### 函数参数

一个函数可以有零个或者多个参数，相同的类型的参数可以简写在一起,例如：上面函数的width和height都是相同的类型，所以可以简写如下

```go
func area(width, height int) int {
	area := width * height
	return area
}
```



#### 函数返回值

与参数一样，函数的返回值也可以是零个或者多个。多余多个返回参数，则没有简写的方式，必须要标明函数返回值的数量和类型，例如计算面积的函数，我们想要返回其周长，则可以改写如下：

```go
func areaAndPerimeter(width, height int) (int, int) {
	area := width * height
	perimeter := 2 * (width + height)
	return area, perimeter
}
```

对于返回参数而言，我们可以直接命名返回参数，就像在函数开始直接声明一样

```go
func areaAndPerimeter(width, height int) (area, perimeter int) {
	area = width * height
	perimeter = 2 * (width + height)
	return
}
```

