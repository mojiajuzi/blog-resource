---
title: "Laravel 队列使用-队列的同步与异步"
date: 2018-05-02T22:42:46+08:00
draft: true
tags: ["Laravel"]
---


关于Laravle所支持的驱动类型以及对应驱动的配置,请查看[Laravel Queue](https://laravel.com/docs/5.6/queues), 在这里将会使用以数据库作为驱动来说明,Laravel队列的使用


#### 准备工作

1. 设置驱动类型,修改`.env`文件中配置选项
    ```ini
    QUEUE_DRIVER=database
    ```

1. 生成任务表并执行迁移
    ```shell
    $ php artisan queue:table

    $ php artisan migrate
    ```
    以上将会生成`jobs`和`failed_jobs`任务表,数据表字段我们随后再做解释

1. 创建两个不同类型任务
   ```shell
    $ php artisan make:job HappybirdJob

    $ php artisan make:job --sync  HappybirdSyncJob
   ```

1. 数据表结构
    ```
    |id|queue|payload|attempts|reserved|reserved_at|available_at|created_at|
    ```
    - queue:队列名称
    - payload:存储序列化之后的job模型
    - attempts:重试次数
    - reserved: 任务是否保留
    - reserved_at: 保留时间
    - available_at:执行时间
    - created_at: 创建时间

#### 结构

1. HappybirdSyncJob

    `App\Jobs\HappybirdSyncJob.php`文件的内容如下
    ```php
    <?php
    namespace App\Jobs;

    use App\Jobs\Job;

    class HappybirdSyncJob extends Job
    {
        public function __construct(){}
        public function handle(){}
    }
    ```

    这个类包含了一个初始化函数,以及一个业务逻辑执行函数,并且继承了`Job`类,查看可以看到其只是使用了一个组合,

    通过查看使用的trait`Queueable`,其内容非常简单,也就是三个方法

    ```php
    // 设置连接的驱动
    public function onConnection(){}

    // 设置队列名称
    public function onQueue(){}

    // 设置连接的时间,单位为秒
    public function delay(){}
    ```

1. HappybirdJob
在`App\Jobs\HappybirdJob.php`文件中,除了基本的初始化函数以及一个执行业务逻辑的函数之外,还使用了`ShouldQueue`这个空接口,并且引入了`InteractsWithQueue, SerializesModels`两个`traits`

    ```php
    class HappybirdJob extends Job implements ShouldQueue
    {
        use InteractsWithQueue, SerializesModels;
       // .....
    }
    ```

    在`InteractsWithQueue`的结构如下

    ```php
    //获取执行的次数
    public function attempts(){}

    // 将任务从队列中删除
    public function delete(){}

    // 手动将任务置换成失败
    public function failed(){}
    
    // 将任务重新放回队列,并设置其延迟执行时间,默认为立即执行
    public function release(int $delay){}

    // 设置任务
    public function setJob(JobContract $job){}
    ```

    `SerializesModels`主要是用来对`Job`模型进行操作,对外值暴露了两个方法
    
    ```php
    // 为实例的序列化做准备,并且返回实例的属性:`connection, queue,delay,job`以及用户自己在任务中设置的属性
     public function __sleep(){}

     // 在任务模型序列化之后,重新存储模型
     public function __wakeup(){}
    ```
    
#### 差异
对于`HappybirdSyncJob`而言,当创建一个任务以后会同步执行,并且任务并不会写入到数据库中,
直到任务创建完成以后,才会返回结果,
而对于`HappybirdJob`而言,将会立即返回结果,任务将会在后台执行
