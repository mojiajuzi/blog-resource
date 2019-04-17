---
title: "Golang-Defer"
description: Defer声明执行函数或者方法的善后工作
date: 2019-04-05T08:27:43+08:00
draft: true
---

defer 声明总是再函数或者方法执行的最后执行，如果函数或者方法有返回值，那就是在返回之前执行

```go
func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
```
以上将会输出
```
hello
world
```
可以看到，它的执行顺序时在函数的最后执行的，那么对于一个函数中包含多个`defer`声明的时候，其执行顺序时怎样地的

```go
defer fmt.Println("world")

fmt.Println("hello")

defer fmt.Println("second defer")
```
以上函数将会输出
```
hello
second defer
world
```
由此可见，对于`defer`来说，其时存储在一个栈中的，当语句块执行到`defer`声明的时候，那么该声明将会背压入栈
由于栈先进后出的特点，那么对于多个`defer`声明而言，那么其执行顺序就清楚了。

当一个`defer`声明执行的时候，它只会纪录当前的状态，而不会受之后执行语句的影响
```go
func main() {
	i := 0
	defer fmt.Println(i)
	i++
	return
}
```
虽然`fmt.Println(i)`虽然在变量`i`的值改变之前执行，但是它所执行的结果时变量递增之前的结果




