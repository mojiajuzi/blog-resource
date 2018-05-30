---
title: "Laravel 事件监听简单使用"
date: 2018-04-27T23:35:21+08:00
draft: true
tags: ["Laravel"]
---


1. 定义事件与事件监听器, 在`App\Providers\EventServiceProvider.php`类的$listen中定义事件与事件监听者的关系

    ```php
        protected $listen = [
            'App\Events\HappybirdEvent' => [
                'App\Listeners\HappybirdListener',
            ],
        ];
    ```

1. 创建事件和事件监听器, 在终端中执行artisan命令来创建

    {{< highlight  bash>}}
    $ php artisan event:generate
    {{< /highlight>}}​

1. 定义事件结构, 在事件类`App\Events\HappybirdEvent`设置事件所需要的条件或者环境
    {{< highlight php>}}
    <?php
    public $item;
    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct(int $item)
    {
        $this->item = $item;
    }
    {{< /highlight>}}
    当然在这里里面,我们可以定义一些处理不同监听者的逻辑,这一点在含有多个监听者的条件下尤其有效

    {{< highlight php>}}
    <?php
    public function addItem(){
        $this->item += 1;
    }
    {{< /highlight>}}

1. 实现监听逻辑, 在`App\Listeners\HappybirdListener`

    {{< highlight php>}}
    <?php
    /**
     * Handle the event.
     *
     * @param  HappybirdEvent  $event
     * @return void
     */
    public function handle(HappybirdEvent $event)
    {
       //Do something
    }
    {{< /highlight>}}

    在监听类的处理句柄`handle`中,实现了事件的依赖注入,这样就可以直接在函数体中获取事件的参数

    {{< highlight php>}}
    <?php
    $event->itme
    {{< /highlight>}}

1. 触发事件, 在你需要的地方,按照如下的方式触发事件
    {{< highlight php>}}
    <?php
    $item = 2;
    event(new HappybirdEvent($item))
    {{< /highlight>}}

#### 注意事项

1. 默认情况下,事件的监听是阻塞的,
也就是说,当代码执行到事件触发的地方将会暂停执行,
直到监听事件完成,才会继续执行后面的业务逻辑, 例如我们可以在监听者中添加如下代码,验证我们的观点

    {{< highlight php>}}
    <?php
    public function handle(HappybirdEvent $event)
    {
      sleep(10);
    }
    {{< /highlight>}}

1. 触发事件监`event`将会返回 `array|null`,如果监听者中没有返回值,将会自动返回null,如果有两个监听者,将会返回两个监听者返回的值

    {{< highlight php>}}
    <?php
    protected $listen = [
        'App\Events\HappybirdEvent' => [
            'App\Listeners\HappybirdListener',
            'App\Listeners\HappybirdListenerB',
        ],
    ];
    {{< /highlight>}}

    {{< highlight php>}}
    <?php
    public function handle(HappybirdEvent $event)
    {
        sleep(5);
        return ["b" => "B"];
    }
   {{< /highlight>}}

    {{< highlight php>}}
    <?php
        public function handle(HappybirdEvent $event)
        {
            return ['a' => 'A'];
        }
    {{< /highlight>}}

执行以上,将会返回如下结果
  
{{< highlight php>}}
<?php
    array:2 [▼
    0 => array:1 [▼
        "a" => "A"
    ]
    1 => array:1 [▼
        "b" => "B"
    ]
    ]
{{< /highlight>}}
