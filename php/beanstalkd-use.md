---
title: "beanstalkd-Laravel Queue"
date: 2018-06-26T15:54:40+08:00
draft: true
---

使用beanstalkd作为Laravel的队列驱动,需要安装[pda/pheanstalk](https://github.com/pda/pheanstalk)包

1. 安装依赖包
    ```
    composer require pda/pheanstalk
    ```

1. 修改`.env`文件中的项目驱动为`beanstalkd`
    ```
    QUEUE_DRIVER=beanstalkd
    ```

1. 调整`config\queue.php`文件中`beanstalkd`数组
    ```php
    <?php
        'beanstalkd' => [
            'driver' => 'beanstalkd',
            'host' => env('BEANSTALKD_HOST', 'localhost'),
            'queue' => 'default',
            'retry_after' => 90,
        ],
    ```

1. 队列的使用文档,可以参考[Laravel Queue](https://laravel.com/docs/5.6/queues)


1. 如果你不使用Laravel的队列和计划任务,[pad/pheanstalk Pheanstalk](https://github.com/pda/pheanstalk/blob/master/src/Pheanstalk.php)类文件对协议中的方法进行封装

1. 安装[beanstalk_console](https://github.com/ptrofimov/beanstalk_console),对beanstalkd中的任务进行管理


#### 参考文档

- [Laravel](https://laravel.com/)

- [pda/pheanstalk](https://github.com/pda/pheanstalk)

- [beanstalk_console](https://github.com/ptrofimov/beanstalk_console)