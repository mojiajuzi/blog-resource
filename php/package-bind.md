---
title: "Laravel 包编写(二)-服务提供者"
date: 2018-06-29T10:31:29+08:00
draft: true
---

#### 创建服务提供者

在Laravel中,我们可以通过`artisan`命令创建一个服务提供者
```
php artisan make:provider HelloServiceProvider
```
以上命令将会创建一个名为`HelloServiceProvider`的服务提供者


#### 服务提供者结构

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class HelloServiceProvider extends ServiceProvider
{
    public function boot(){
        // TODO
    }

    public function register(){
        //TODO
    }
}
```

在Laravel中,服务提供者继承了`Illuminate\Support\ServiceProvider`类,并且包含两个方法`boot`和`register`

- **register:** 该方法中我们要做的就是一件事,那就是将服务绑定到容器中,
对于其他的诸如事件监听,路由或者其他的服务则不应该放到这里,
这是由于该方法的调用是在服务的注册阶段,其他的服务提供者可能还未完成注册


- **boot:** 我们将可以使用其他服务提供者提供的服务,
因为该方法的调用是在所有其他的服务提供者全部注册完成之后,因此,我们可以使用框架中的所有其他的服务


- **ServiceProvider:** 在这个类的初始化方法中就是注入应用的实例`$app`,
对于我们的服务提供这而言基本会涉及到的东西就是: 配置文件,路由,控制器,数据模型,数据迁移等,因此`ServiceProvider`提供了便捷的方法
方便我们实现以下内容

    - `mergeConfigFrom($path, $key)` 配置文件的加载

    - `loadRoutesFrom($path)` 加载路由

    - `loadViewsFrom($path, $namespace)` 加载视图

    - `loadTranslationsFrom($path, $namespace)`, `loadJsonTranslationsFrom`加载多语言

    - `loadMigrationsFrom($paths)` 数据迁移

    - `publishes(array $paths, $group = null)` 发布

    - `commands($commands)` 注册`Artisan`命令

    - `isDeferred()` 延迟注册

#### 注册服务提供者

注册服务提供者,只需要将服务提供者添加到`config/app.php`文件的`providers`数组中即可
```
App\Providers\HelloServiceProvider::class,
```


#### 参考文档
- [Service Providers](https://laravel.com/docs/5.6/providers)



