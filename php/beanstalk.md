---
title: "beanstalkd-基本概念"
date: 2018-06-22T17:13:23+08:00
draft: true
---

在beanstalkd中包含几个重要的概念，如下图所示，对于一个系统消息队列系统来说
其必定有
- 一个或多个生产者(Producter),
- 一个或多个消费者(Customer),
- 一个消息存储系统，在这里指的是beanstalkd

对于beanstalkd来说，以管道(Tube)作为其主要的组成部分，一个beanstalkd可以包含一个或者多个管道

![beanstalkd](/images/beanstalkd-systerm.png)



对于Tube而言,有两个队列组成，`ready queue`和`delay queue`,每一个队列里面可以包含零个或者多个任务

- `ready` 队列用来存储满足执行条件的队列
- `delay` 队列用来存储尚未到执行时间的队列

![beanstalkd](/images/beanstalkd-tube.png)


对于队列中的任务而言，我们更加关注的是任务的状态，因此在`beanstalkd`中的任务包含一下四中状态
`ready, reserved, delayed,buried`

- `ready` 需要立即被处理的任务
- `reserved` 已经被消费者获取，正在处理的任务
- `delayed` 延迟执行的任务
- `buried` 已经被执行完，但是未删除的任务

![beanstalkd](/images/beanstalkd-stats.png)


1. 由上面的流程图可以看到，生成任务时，可以将任务设置成立即执行(ready)或者延迟执行(delayed)

1. 对于延迟任务而言，延迟时间到了以后，其状态变更为立即执行状态

1. 只有在立即执行状态下的任务，才能被消费者消费

1. 处于立即执行状态下的任务，被消费者获取以后，其状态变成正在处理中(reserved)

1. 当任务处理完以后，有四种处理方式删除，重新投递到立即执行队列，投递到延迟队列以及不做任何处理

1. 当不对已完成任务做任何处理时，其对消费者属于不可见状态，直到其被重新投递到`ready`队列中，或者被删除

```asciiarmor
   put with delay               release with delay
  ----------------> [DELAYED] <------------.
                        |                   |
                        | (time passes)     |
                        |                   |
   put                  v     reserve       |       delete
  -----------------> [READY] ---------> [RESERVED] --------> *poof*
                       ^  ^                |  |
                       |   \  release      |  |
                       |    `-------------'   |
                       |                      |
                       | kick                 |
                       |                      |
                       |       bury           |
                    [BURIED] <---------------'
                       |
                       |  delete
                        `--------> *poof*
```
