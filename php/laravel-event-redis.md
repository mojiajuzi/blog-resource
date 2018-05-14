---
title: "Laravel 队列使用-Redis作为驱动"
date: 2018-05-07T22:23:05+08:00
draft: true
tags: ["Laravel"]
---

#### 准备工作

1. 安装相对应的Redis包
    ```bash
    $ composer require  composer require predis/predis
    ```

1. 修改配置文件`.env`,设置驱动
    ```ini
    QUEUE_DRIVER=redis

    REDIS_HOST=redis
    REDIS_PASSWORD=null
    REDIS_PORT=6379
    ```

1. 修改`config/queue.php`文件中redis队列连详情
    ```php
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default',
            'queue' => 'default',
            'expire' => 60,
        ],
    ```


1. 设置任务运行失败是的存储,失败的任务是存放到数据库表中的,在`config/queue.php`文件中配置相关的选项
```php
    'failed' => [
        'database' => env('DB_CONNECTION', 'mysql'),
        'table' => 'failed_jobs',
    ],
```

#### 任务存储结构
在Redis中,队列根据是否延迟执行分为两种结构

#####  无延迟队列    

无延迟队列在Redis中的存储结构为`Lists`,其key的命名规则以`queues`关键字后面添加队列名称

```
queues:happybird
```

##### 延迟队列
延迟队列由于引入了延迟执行的时间概念,所以其存储的结构为`Sort Sets`, 其key的命名规则以`queues`关键字后面添加队列名称以及`delayed`关键字
```
queues:happybird:delayed
```

执行时间为`Sort Sets`的`score`