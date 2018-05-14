---
title: "Laravel 队列使用-队列的触发与守护"
date: 2018-05-03T23:07:15+08:00
draft: true
tags: ["Laravel"]
---


#### 触发

任务的触发,主要的实现是在`Illuminate\Foundation\Bus\DispatchesJobs`这个trait中实现的,其只包含两个方法
```php
protected function dispatch($job){
    return app(Dispatcher::class)->dispatch($job);
}

protected function dispatchNow($job){
    return app(Dispatcher::class)->dispatchNow($job);
}
```
这两个方法的区别就在于时间上,一个是推送到队列,另外一个是推送并立即执行,依据前面的`traitQueueable`可知,我们可以在对任务进行设置,例如

```php
protected function hpJob(){
    $job = (new HappybirdJob(5))->onConnection("other")
        ->onQueue("happybird")
        ->delay(5);
}
```
以上例子表示,我们使用`other`的连接在`happybird`队列中设置一个延迟时间为`5s`的`HappybirdJob`任务

由于`dispatch, dispatchNow`都是使用的是容器中绑定的`Dsipatch`类,接下来看一下这个类
这个类位于`Illuminate\Bus\Dispatcher`中

##### Dispatch
该类主要是用于队列任务的分发以及设置,其中可以看到

```php
    public function dispatch($command)
    {
        if ($this->queueResolver && $this->commandShouldBeQueued($command)) {
            return $this->dispatchToQueue($command);
        } else {
            return $this->dispatchNow($command);
        }
    }
```
当使用`dispatch`类触发任务时,将需要判断是否实现了`ShouldQueue`,这个就是之前在讨论异步队列和同步队列时两个不同的列所实现的差异

而对于立即执行的操作来说,则是通过管道来执行
```php
    public function dispatchNow($command)
    {
        return $this->pipeline->send($command)
            ->through($this->pipes)
            ->then(function ($command) {
                return $this->container->call([$command, 'handle']);
        });
    }
```
对于管道的说明和解释,可以参考
[Laravel Pipeline 组件的实现](https://www.insp.top/article/realization-of-pipeline-component-for-laravel),
[Understanding Laravel Pipelines](https://medium.com/@jeffochoa/understanding-laravel-pipelines-a7191f75c351)这两篇文章


#### 执行
对于推送到队列里面的任务,可以通过`artisan`命令及其参数来进行控制,通过如下命令查看详情
```
$ php artisan list | grep queue
```

将可以得到如下的结果:
```
  queue:failed        List all of the failed queue jobs
  queue:failed-table  Create a migration for the failed queue jobs database table
  queue:flush         Flush all of the failed queue jobs
  queue:forget        Delete a failed queue job
  queue:listen        Listen to a given queue
  queue:restart       Restart queue worker daemons after their current job
  queue:retry         Retry a failed queue job
  queue:table         Create a migration for the queue jobs database table
  queue:work          Process the next job on a queue
```
这里需要特别注意的是,`queue:work, queue:listen`这两个命令,其余的都没有额外的参数

##### queue:listern

```
--queue[=QUEUE]      The queue to listen on
--delay[=DELAY]      Amount of time to delay failed jobs [default: 0]
--memory[=MEMORY]    The memory limit in megabytes [default: 128]
--timeout[=TIMEOUT]  Seconds a job may run before timing out [default: 60]
--sleep[=SLEEP]      Seconds to wait before checking queue for jobs [default: 3]
--tries[=TRIES]      Number of times to attempt a job before logging it failed [default: 0]
```

##### queue:work
```
--queue[=QUEUE]    The queue to listen on
--daemon           Run the worker in daemon mode
--delay[=DELAY]    Amount of time to delay failed jobs [default: 0]
--force            Force the worker to run even in maintenance mode
--memory[=MEMORY]  The memory limit in megabytes [default: 128]
--sleep[=SLEEP]    Number of seconds to sleep when no job is available [default: 3]
--tries[=TRIES]    Number of times to attempt a job before logging it failed [default: 0]
```

相同:

- queue: 当包含多个队列时,指定其执行的优先级,优先级顺序为:越往前优先级越高
- delay: 失败队列的延迟执行时间
- memory: 任务执行的最大使用内存,默认为128M
- tries: 任务失败后,尝试运行的次数,默认为不启动

差异:
对于`listen`来说,其包含一个`--timeout`参数,用来置顶执行的时间,超过该时间,将会造成任务失败,程序异常,队列停止

而`--sleep`选项对于`listen`来说,表示暂时未有任务执行时,重新执行任务需要的时间,对于`queue:work`来说,则表示当没有任务运行时,重新检测任务的时间间隔

`queue:work`还有一项特别的参数就是用来指定`work`运行的模式,如果启用,则会强制队列服务器持续处理任务,无需重启框架


#### 守护
官方推荐使用[Supervisor](http://supervisord.org/index.html)来守护队列的进程,其推荐的配置如下:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --daemon
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
```

- command:用来设置运行的命令,这个需要替换成自己的环境配置
- numprocs: 监控的进程数量,需要注意的是,进程的数量不能设置过大
