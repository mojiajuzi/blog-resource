---
title: "Golang-Panic & Recover"
date: 2019-04-09T20:37:33+08:00
description: "通常情况下应该避免使用panic和recover,而是应该使用error,只有在程序无法继续执行的时候才使用"
draft: true
---

### panic

panic: 终止程序继续执行, 当在一个函数中使用panic之后,将会执行一下步骤：

1. 程序将会停止执行
1. 执行`defer`定义的语句
1. 返回给调用该函数的地方
1. 进程继续执行，直到在该goroutine中的所有函数全部返回
1. 打印出panic堆栈信息
1. 进程终止执行

```go
func main() {
	defer fmt.Println("end")
	go hello()
	time.Sleep(2 * time.Second)
	fmt.Println("main")
}

func hello() {
	defer fmt.Println("world")
	panic("this is error")
	fmt.Println("hello")
}
```
以上会打印出如下结果，验证了上面的执行顺序
```
world
panic:
this is error

goroutine 19 [running]:
main.hello()
	C:/code/go/src/hello/main.go:18 +0xc5
created by main.main
	C:/code/go/src/hello/main.go:11 +0xdf
```

### recover
recover接管并处理panic，具有以下几个特点:

1. 只有在使用`defer`函数时，recover才有用。
1. 只能接管相同goroutine内的panic

```go
func hello() {
	defer world()
	panic("this is error")
	fmt.Println("hello")
}

func world() {
	if r := recover(); r != nil {
		fmt.Println(r)
	}
}

```
以上结果打印出:
```go
this is error
main
end
```
由此可见。recover接管了panic的错误，并且保证了该启用goroutine的goroutie能够继续执行

为了打印堆栈信息，可以使用`runtime/debug`这个包
```go
func world() {
	if r := recover(); r != nil {
		fmt.Println(r)
	}
	debug.PrintStack()
}
```
将会打印如下结果:
```
this is error
goroutine 19 [running]:
runtime/debug.Stack(0x0, 0x0, 0x0)
	C:/app/Go/src/runtime/debug/stack.go:24 +0xa8
runtime/debug.PrintStack()
	C:/app/Go/src/runtime/debug/stack.go:16 +0x29
main.world()
	C:/code/go/src/hello/main.go:27 +0xb9
panic(0x4bf940, 0x4f1a00)
	C:/app/Go/src/runtime/panic.go:522 +0x1ee
main.hello()
	C:/code/go/src/hello/main.go:19 +0x5e
created by main.main
	C:/code/go/src/hello/main.go:12 +0xdf
main
end
```