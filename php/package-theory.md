---
title: "Laravle 包编写(一)-Laravel启动过程"
date: 2018-06-29T09:37:49+08:00
draft: true
---

在Laravel文档[请求的生命周期](https://laravel.com/docs/5.6/lifecycle)一篇中,讲解了请求的生命周期

1. 首先请求进入`public/index.php`文件中, 该文件首先加载`autoload.php`文件,用于类的自动加载
    ```
    require __DIR__.'/../vendor/autoload.php';
    ```

1. 然后引入应用的初始文件,并返回一个应用的实例`$app`
    ```
    $app = require_once __DIR__.'/../bootstrap/app.php';
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
    ```

1. 在初始化`$app`实例的初始化函数中,进行了如下的操作
    ```
    public function __construct($basePath = null)
    {
        if ($basePath) {
            $this->setBasePath($basePath);
        }

        // 实例化容器
        $this->registerBaseBindings();

        // 注册基础服务(事件监听服务, 日志服务, 路由服务)
        $this->registerBaseServiceProviders();

        // 注册核心容器的别名
        $this->registerCoreContainerAliases();
    }
    ```

1. 在实例化`Illuminate\Foundation\Application`之后,使用单例模式将http请求的核心注册到容器中
    ```
    // http请求核心
    $app->singleton(
        Illuminate\Contracts\Http\Kernel::class,
        App\Http\Kernel::class
    );

    // 命令行请求核心
    $app->singleton(
        Illuminate\Contracts\Console\Kernel::class,
        App\Console\Kernel::class
    );

    //异常处理
    $app->singleton(
        Illuminate\Contracts\Debug\ExceptionHandler::class,
        App\Exceptions\Handler::class
    );
    ```

 1. 调用`Http\Kernel`的`handle`方法将http请求传递
    ```
    $response = $kernel->handle(
        $request = Illuminate\Http\Request::capture()
    );
    ```

 1. 在`handle`方法中,会对请求进行处理
    ```
    $request->enableHttpMethodParameterOverride();
    $response = $this->sendRequestThroughRouter($request);
    ```

1. 在调用`sendRequestThroughRouter`方法的时候,调用了`$this->bootstrap();`这个方法,而这个方法的主要目的是加载定各种服务服务
    ```
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ```

1. 在这里我们特别关注的就是`\Illuminate\Foundation\Bootstrap\RegisterProviders::class,`这一行,这里是加载`app\Provider`中自定义服务提供者的关键地方


1. 然后就是对请求使用`Pipeline`进行处理,最终返回响应

 
由于我们的包本质上是提供一种服务,根据上面的流程分析,我们要做的就是将包以服务提供者的身份注册到容器中,也就是上面第8个步骤中

