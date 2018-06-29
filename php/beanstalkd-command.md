---
title: "beanstalkd-协议"
date: 2018-06-26T12:06:38+08:00
draft: true
---

beanstalk协议以ASCII编码运行在TCP协议上；客户端的执行的周期为：连接服务，发送命名和数据，等待响应，关闭连接，对于一个连接而言；服务器按接收顺序依次处理命令，并以相同的顺序发送响应；所有的数字都将转换成无符号的十进制整型．


命名，对于ASCII字符串而言，名字可以包含字母(A-Z, a-z), 数字(0-9),横线('-'),加号('+'), 斜线("/"), 分号(";"), 顿号("."), 美元符号("$"),以及括号("()")，但是不能以横线作为开头．字符串以空白符结尾，但是每一个名字至少包含一个字符

该协议包含两种数据格式，文本行，非结构化的数据块．其中文本行主要用于客户端命令和服务端响应，数据块常用来保存任务详情以及状态．每一个消息体
都是一个字节序列，服务端不会对消息进行检查和修改，只会原样返回,这样使得客户端能够正确的解析消息

beanstalk中并没有用于关闭连接的命令，客户端如果长时间未使用服务将会自动关闭TCP连接，对于beanstalk而言，能够同时保持大量的连接，对于客户端而言就能够更好的保持连接以及重用连接，这样就避免了创建新的TCP连接带来的额外开销

如果客户端违反协议(如:发送非法格式请求数据,命令不存在)或者服务端发生了错误，客户端将会返回如下的错误信息

    - "OUT_OF_MEMORY\r\n" 内存不足，服务端无法分配足够的内存用于消息的执行，客户端需要等待一段时间再尝试发送
    
    - "INTERNAL_ERROR\r\n" 内部错误，服务端出现了BUG
    
    - "DRAINING\r\n" 服务端不再接受新的消息，客户端需要尝试连接其他的服务或者关闭服务
    
    - "BAD_FORMAT\r\n" 客户端发送了错误的数据格式
    
    - "UNKNOWN_COMMAND\r\n"　客户端发送了错误的命令


#### 生产者命令

指定使用的Tube,如果不指定Tube,那么任务将会被投递到一个名为`default`的Tube中,其名称长度不得大于200bytes,如果Tube不存在将会新建一个,执行成功后,将会返回`USING <tube>\r\n`

```
use <tube>\r\n
```

创建一个新的任务

```
put <pri> <delay> <ttr> <bytes>\r\n
```
- pri 指定任务的优先级,0-2**32

- delay 任务的延迟执行时间,单位为秒,如果指定,则任务将会被投递到Tube的延迟队列中

- ttr 允许消费者处理任务的时间,单位为秒,该时间从消费者获取任务后开始计时,在该时间区间内
如果消费者不变更任务的状态或者删除任务,那么该任务将会被重新投递到`ready queue`中,该值的最小值为1,如果设置为0,那么服务端将会自动增加为1

- bytes 任务的编码大小,任务编码以后,其大小不得超过2**16 bytes

- data 消息体

创建一个任务的可能响应值如下

- `INSERTED <id> \r\n` 任务写入成功，并返回任务的`id`标识

- `BURIED <id>\r\n` 优先级队列已经耗尽内存，任务状态将会被设定为`BURIED`

- `EXPECTED_CRLF\r\n` 消息体没有结束符号`\r\n`

- `JOB_TOO_BIG\r\n` 消息体大小超过限制

#### 任务操作命令

消费者通过使用`reserve`命令从Tube的`ready queue`队列中获取可执行的任务

```
reserve\r\n

reserve-with-timeout <seconds>\r\n
```
如果`ready queue`队列中,没有可用的任务,则会等待有可用的任务时才返回响应,一旦
获取到任务,那么就需要在之前创建任务时指定的`<ttr>`时间间隔内对任务进行处理,一旦超时
则会将任务重新投递到`ready queue`队列,超时时间和剩余时间可以通过`stats-job`命令获得


最后一秒由服务器保存为安全边际，在此期间客户不会被迫等待另一个任务,如果消费者对任务执行了
`reserve`操作,或者时间已到,服务端将会做出如下的反应

- `DEADLINE_SOON\r\n` 在服务端自定自动将任务放到`ready`队列之前，客户端还有最后一次机会对任务进行处理(删除或者放到`ready`队列)

- `TIME_OUT\r\n` 超时时间结束时，队列中没有任务执行，将会返回超时

正常情况下,获取到的任务响应结构如下

```
    RESERVED <id> <bytes>\r\n
    <data>\r\n
```


任务执行完成以后,可以使用`delete`命令,将任务删除

```
delete <id>\r\n
```
执行删除后，可能的响应为
- `DELETED\r\n` 删除成功

- `NOT_FOUND\r\n` 任务不存在

当然,也可以通过使用`release`命名,将任务重新投递到队列中,该命令通常用于任务处理失败的情况

```
release <id> <pri> <delay> \r\n
```
在任务由`reserved`状态变更为`ready`或者`delayed`状态的过程中,其可能会返回如下的响应

- `RELEASED\r\n` 执行成功

- `BURIED\r\n` 执行成功，由于内存不足，任务将会被隐藏

- `NOT_FOUND\r\n` 任务无法找到

对于执行完成的任务而言,可以使用`bury`命令来将任务保留,知道将其重新投递到`ready queue`队列或者删除

```
bury <id> <pri>\r\n
```

执行成功以后,可能会返回的响应如下

- `BURIED\r\n` 执行成功

- `NOT_FOUND\r\n` 任务不存在

通过使用`kick`命令,可以手动的将`buried`状态的任务变更为`ready`状态

```
kick <bound>\r\n
```
- `bound`所移动的最大任务数量

执行后,将会返回对应的执行成功的任务数量`KICKED<count>\r\n`


对于耗费时间较长的任务,除了在创建时设置执行时间,还可以通过`touch`来更新任务的执行时间

```
touch <id>\r\n
```
执行成功后,返回的响应可能为

- `TOUCHER\r\n` 执行成功

- `NOT_FOU\r\n` 任务不存在或者任务尚未执行

#### 消费者命令

对于消费者而言,可以通过`watch`命令,来增加消费者的消费Tube,执行成功后,将会返回连接的Tube数量`<count>\r\n`, 

```
watch <tube>\r\n
```
相反的,可以通过`ignore`命令来将Tube从列表中剔除
```
ignore <tube>\r\n
```
剔除后,可能会返回如下的响应

- `WATCHING <count>\r\n`

- `NOT_IGNORED\r\n` 如果列表中只有一个管道,将会返回该值


对于一个任务而言,有时候需要获取该任务的详细信息,因此通过`stats-job`命令可以查看任务在当前状态下信息
```
stats-job <id>\r\n
```
那么,其可能的响应为以下两种情况

- `NOT_FOUNDr\n` 

- `OK <bytes>\r\n<data>\r\n`

对于任务存在的时候,返回的`data`将会是一个YAM格式的数据,其包含如下的信息

- id 任务标识
- tube 管道名称
- state 任务状态
- pri 优先级
- age 从使用`put`命令生成任务后开始计算的时间,单位为秒
- time-left 当服务器将该任务投递到`ready queue`队列时开始计算的时间,该时间只在任务处于`reserved`和`delayed`有效
- timeouts 任务处理时间
- releases 客户端获取任务执行已花费的时间
- buries 任务被保留的时间
- kicks 保留任务被重新投递到`ready queue`的时间


当然Tube的状态也是一个我们需要关注的点,通过使用`stats-tube`命令,可以获取某个Tube的当前状态
```
stats-tube <tube>\r\n
```
那么,其可能的响应为以下两种情况

- `NOT_FOUNDr\n` 

- `OK <bytes>\r\n<data>\r\n`

对于Tube存在的时候,返回的`data`将会是一个YAM格式的数据,其包含如下的信息
 
 - "name" Tube名称.

 - "current-jobs-urgent" 任务优先级小于1024级别的任务列表

 - "current-jobs-ready" 可执行的任务列表

 - "current-jobs-reserved" 正在执行的任务列表

 - "current-jobs-delayed" 延迟执行的任务列表.

 - "current-jobs-buried" 隐藏的任务列表

 - "total-jobs" 任务记录总和

 - "current-waiting" 等待响应的消费者列表

更进一步,通过使用`stats`命令可以获取整个系统的统计信息,这里不在描述,直接看说明文档


其他的一些统计命令如下:

- `list-tubes\r\n` 可以获取系统中所有的Tube名称列表

- `list-tube-users\r\n` 返回当前正在被客户端使用的Tube列表

- `list-tubes-watched\r\n` 返回正在被连接的Tube列表 


#### 参考资料

- [beanstalk doc](https://github.com/kr/beanstalkd/blob/master/doc/protocol.txt)
