---
title: "Laravel 队列使用-队列失败任务处理"
date: 2018-05-04T21:57:38+08:00
draft: true
tags: ["Laravel"]
---

在`config/queue.php`文件的`failed`字段里面,可以配置失败任务的连接以及数据的存储

```php
    'failed' => [
        'database' => env('DB_CONNECTION', 'mysql'),
        'table' => 'failed_jobs',
    ],
```

在使用`artisan`命令来执行队列任务的时候,如果没有指定任务失败的次数,那么任务将会一直尝试
例如在`HappybirdJob`类的`handle`函数中添加如下代码
```php
    public function handle()
    {
        $data = [];
        $data[3];
    }
```
当执行的时候,将会一直不断的尝试执行而不会停止,如果指定了`--tries`次数,那么任务将会在执行指定的次数之后删除任务并将任务放置到失败的任务表中
> Note: 任务序列化存储的时候只会存储对应的上下文环境,但是不包括handle中的业务逻辑,所以测试的时候需要注意这一点


#### 注册失败任务监听任务

在`App\Providers\AppServiceProvider`这个文件的`AppServiceProvider`类中引入如下类:

```php
use Illuminate\Support\Facades\Queue;
use Illuminate\Queue\Events\JobFailed;
```

然后在`boot`中做如下处理:
```php
    Queue::failing(function(JobFailed $event){
        // deal with the fail job
    });
```

通过查看`JobFailed`可以看到,这个类没有任务方法,仅仅只是保存了任务的一下几个属性:

```php
public function __construct($connectionName, $job, $data, $failedId = null)
{
    $this->job = $job; //任务对象
    $this->data = $data; // 传递给任务的数据
    $this->failedId = $failedId; // 失败任务记录ID
    $this->connectionName = $connectionName; //队列连接名称
}
```
通过`job`对象,我们就可以获取到任务的所有详情,然后就可以针对任务进行处理

####  在任务类中处理

当任务失败的时候,可以在任务类中实现`failed`方法,这样就可以对失败的任务进行处理
```php
public function failed(){
    // deal with the fail job
}
```

