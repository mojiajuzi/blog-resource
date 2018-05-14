---
title: "Goroutine"
date: 2018-02-13T21:52:57+08:00
draft: true
tags: ["Go"]
---

## Goroutines

> Goroutine是由Go runtime管理的轻量级线程，开启一个新的Goroutine只需要使用`go`关键字，由于Goroutine运行在相同的地址空间，因此需要通过共享内存实现同步，包`sync`提供了基本的同步操作

## Channels

> Channel是一种管道类型，使用它可以通过箭头操作符(`<-`)发送和接受数据

默认情况下通过Channel发送、接收是阻塞的，也就是说，Channel的容量是1，举例来说，假设你通过channel发送一个值，如果另外一端不读取数据，那么再次发送数据就是阻塞的，必须等到另外一端接受数据后才能继续发送数据，该机制能够保证数据的同步不需要依赖于显示锁或者是状态值

{{< highlight golang >}}
func main() {
    c := make(chan int)
    for i := 0; i < 3; i++ {
        signCapChan(i, c)
    }
    y := <-c
    fmt.Println(y)
}
func signCapChan(x int, ch chan int) {
    ch <- x
}
{{< /highlight>}}

上面的程序将会提示`all goroutines are asleep - deadlock!`

为了解决这个容量问题，我们需要给channel引入缓存的概念，直接在初始化的时候，给定其容量的大小，因此上面的问题可以如下解决

{{< highlight golang >}}
....
c := make(chan int, 3)
....
{{< /highlight>}}

按照上面的调整以后，程序可以正常运行，但是如果将channel的容量变更为2,依旧会出现：`all goroutines are asleep - deadlock!`的错误,原因在于,对于一个有缓存的channel来说，当往其中写入数据的时候，如果容量满了以后，依旧会发生阻塞，在读取的情况下，如果channel为空的情况下依旧会发生阻塞

{{< highlight golang >}}
...
y := <-c
for i := 0; i < 3; i++ {
    signCapChan(i, c)
}
fmt.Println(y)
...
{{< /highlight>}}

当然使用channel中的缓存在数据量比较小的时候还是非常有用的，如果数据量比较大的情况下，那么依旧采用设置缓存的方式将是不可取的，对于这个问题，以后再说明，先来理解channel的遍历和关闭

### 遍历

对于容度大于1的channel来说，发送到channel中的数据是一个类似于队列，因此要获取channel中的所有数据，那么就需要使用遍历

{{< highlight golang >}}
...    
c := make(chan int, 3)
for i := 0; i < 3; i++ {
    signCapChan(i, c)
}
for ch := range c {
    fmt.Println(ch)
}
...
{{< /highlight>}}

### 关闭

当不再发送数据到channel中时，发送者(发送数据到channel的对象)能够使用`close`方法显式的关闭channel，而一个接受者（从channel中获取数据的对象），能够通过如下方式检测channel是否被关闭

{{< highlight golang >}}
v, ok := <- ch
{{< /highlight>}}

如果`ok`接收的值为`false`时，表明channel已经被关闭了，对于channel的关闭，只能由发送者进行关闭，如果接受者试图关闭channel将会触发错误,与文件不一样的是通常情况下并不需要手动的去关闭channel，仅仅在需要告诉接受者不再发送数据的时候



可以使用`Select`声明来等待一个Goroutine中多个通信操作的完成,例如如下的[*斐波那契数列*](https://baike.baidu.com/item/%E6%96%90%E6%B3%A2%E9%82%A3%E5%A5%91%E6%95%B0%E5%88%97)的计算

{{< highlight golang >}}
func main() {
    c := make(chan int)
    quit := make(chan int)
    //开启一个goroutine
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<-c)
        }
        quit <- 0
    }()
    //等待Goroutine的完成
    fibonacci(c, quit)
}

func fibonacci(c, quit chan int) {
    x, y := 0, 1
    for {
        select {
        case c <- x:
            x, y = y, x+y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}
{{< /highlight>}}

分析：

1. 先声明两个channel
2. 开启一个goroutine
3. 创建函数，使用select声明来等待计算的结束

以上充分使用channel的阻塞以及select声明的特性(满足case条件就执行对应的语句块)，不过这里有如下几点需要注意：

select必须要有一个结束，否则会死锁，比如将如下语句注释掉

{{< highlight golang >}}
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<-c)
        }
    // quit <- 0
    }()
{{< /highlight>}}

那么将会造成一个死锁,因为`c <- x`永远为true，`select`无法结束，造成一个死锁，在这种情况下，我们可以进行一下改造

{{< highlight golang >}}
select {
case c <- x:
    x, y = y, x+y
case <-quit:
    fmt.Println("quit")
    return
default:
    fmt.Println("default")
    return
}
{{< /highlight>}}

在这种情况下，由于主goroutine(main函数所在的goroutine)会先执行完成，只会打印出`default`，因此我们可以取消`quit`,这样我们就可以获得这样的代码

{{< highlight golang >}}
func main() {
    c := make(chan int)
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<-c)
        }
        close(c)
    }()

    fibonacci(c)
}

func fibonacci(c chan int) {
    x, y := 0, 1
    for {
        select {
        case c <- x:
            x, y = y, x+y
        case _, ok := <-c:
            if !ok {
                return
            }
        }
    }
}
{{< /highlight>}}

在这里我们利用了channel的关闭检测，当检测到channel关闭后，然后就退出循环