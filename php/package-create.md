---
title: "Laravel 包编写(三)-服务提供者实现"
date: 2018-06-29T11:32:11+08:00
draft: true
---

在这一篇我们将实现一个简单的关于文章分类的包(`gru\tag`),该包将会尽量涉及到包开发的各个方面

1. 在一个Laravel项目的根目录下面创建一个名为`packages`, 
然后在package目录下面创建一个`gru`文件夹,
最后在`gru`文件夹下面创建一个`tag`文件,创建完成以后,结构如下
    ```
    -Root
        --gru
            --tag
              --src
    ```
    之所以这样布局是因为相同作者的包,使用composer管理以后将统一放到一个目录下,而`src`将会作为我们代码放置的地方


1. 在tag目录下,使用composer初始化包,初始化以后我们就可以得到一个composer.json文件,由于该项目没有依赖其他的包,所以这里添加依赖
    ```json
        {
            "name": "gru/tag",
            "description": "this is demo about package write",
            "type": "library",
            "license": "MIT",
            "require": {}
        }
    ```
    如果不熟悉,composer可以参考[composer中文网](https://www.phpcomposer.com/)提供的文档

1. 模拟自动加载,为了更好的在Laravel项目中进行包的开发与测试,我们需要在项目的根目录(注意:不是tag下面)下面添加命名psr-4的加载规则
    ```json
        "psr-4": {
            "App\\": "app/",
            "Gru\\Tag\\": "packages/gru/tag/src"
        }
    ```
    添加完成以后需要执行`composer dumpautoload`来使配置生效,生效后,我们包的命名空间就会变成`Gru\Tag`

1. 创建服务提供者`GruTagServiceProvider`
    ```
    php artisan make:provider GruTagServiceProvider
    ```
    创建完成以后,需要将该文件迁移到`packages/gru/tag/src`目录下,并修改其命名空间为`Gru\Tag`

1. 将创建的服务注册到`config/app.php`文件的`providers`数组中
    ```
    Gru\Tag\GruTagServiceProvider::class,
    ```

1. 创建数据迁移文件
    ```
    php artisan make:migration create_table_tags --create=tags
    ```
    然后将文件迁移到我们包的`src`目录下,

1. 创建模型,并迁移到`src`目录下,注意变更模型的命名空间
    ```
    php artisan make:model Tag
    ```

1. 创建资源控制器,并迁移到`src`目录下,注意变更模型的命名空间
    ```
    php artisan make:controller --resource TagController
    ```
    这里需要注意的是,处理变更命名空间,还需要引入基础控制器类
    ```
    use App\Http\Controllers\Controller;
    ```

1. 添加路由文件`src\routes.php`
    ```php
    <?php

    Route::resource("tags", "Gru\Tag\TagController");
    ```
1. 添加路由文件到`GruServiceProvider.php`文件的`boot`方法中
    ```php
    <?php
    public function boot()
    {
        $routeFile = __DIR__ . '/routes.php';
        $this->loadRoutesFrom($routeFile);
    }
    ```

1. 在`TagController`类中的`index`方法中,添加如下代码,测试当前路由几控制器是否可用
   ```php
   <?php
     public function index()
    {
        return response()->json(['state' => 'route, controller work']);
    }
   ```

1. 通过路由访问`you_domain/tags`,如果看到
    ```json
    {"state":"route, controller work"}
    ```
    则表明,路由和控制器已经正常工作

1. 在`src`目录下创建配置文件`tag.php`,并添加一点内容
    ```php
    <?php
    return [
        'tag' => 'this is tag config'
    ];
    ```
1. 在`GruServiceProvider`的boot方法中加载配置
    ```php
    <?php
    public function boot()
    {
        .....

        $configFile = __DIR__ . '/tag.php';
        $this->mergeConfigFrom($configFile, 'tag');
    }
    ```
    然后就可以在任何地方通过`config(tag.xx)`的形式进行访问配置文件

1. 执行迁移文件, 在`GruServiceProvider`类的`boot`方法中,增加迁移文件的路径
    ```php
    <?php
        //加载迁移文件
        $migratePath = __DIR__ . '/';
        $this->loadMigrationsFrom($migratePath);
    ```
    然后在项目的根目录下执行迁移
    ```
    php artisan migrate
    ```

1. 视图的使用,在`src`目录下,增加增加一个名为`create.blade.php`的文件,在这里将用来创建tag
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>Tag create</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
    </head>
    <body>
        <form action="{{url('tags')}}" method="post">
            <input type="text" name="name">
            <input type="submit" value="提交">
        </form>
    </body>
    </html>
    ```
1. 在`GruServiceProvider`类的`boot`方法中,添加视图加载路径
    ```php
    <?php
        //加载视图文件
        $viewPath = __DIR__ . '/';
        $this->loadViewsFrom($viewPath, 'Gru\Tag');
    ```

1. 然后在控制的`create`方法中,增加视图的返回
    ```php
    <?php
    public function create()
    {
        return view('Gru\Tag::create');
    }
    ```
    在这里需要注意的是,这里的视图加载需要使用到`loadViewsFrom`中的命名空间来指定视图的地址

1. 最后在控制器的`store`方法中,添加标签保存的逻辑处理,这里为了简化操作,去除了必要的校验处理
    ```php
    <?php
    public function store(Request $request)
    {
        $name = $request->get("name");
        $tag = new Tag;
        $tag->name = $name;
        $tag->save();
    }
    ```

至此一个包的创建过程基本就完成了,由于这里我们没有提供特别的服务类,
因此没有在`register`中注册服务,如果有的话,注册的代码如下:

```php
<?php
// 服务注册
$this->app->singleton(MyPackage::class, function () {
    return new MyPackage();
});

// 别名注册
$this->app->alias(MyPackage::class, 'my-package');
```

#### 不足

1. 一个完整的包,离不开测试,这里我们没有进行代码的测试,在实际的包开发中,应该补充包测试,以及使用文档

1. 这里我们为了方便,没有对包的目录结构进行设计,实际上应该依据自己的需要,调整包各个模块的结构







#### 参考资料
- [Getting started on Laravel package development](https://medium.com/@lasselehtinen/getting-started-on-laravel-package-development-a62110c58ba1)

- [How to create a Laravel Package](https://devdojo.com/blog/tutorials/how-to-create-a-laravel-package)