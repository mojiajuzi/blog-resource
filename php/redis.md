---
title: "Laravel中安装和使用Redis作为会话和缓存的载体"
date: 2018-03-08T23:27:06+08:00
draft: true
tags: ["Laravel"]
---


> 本人使用的是[Laradock](http://laradock.io/) 作为开发环境

### predis的安装

1. 安装依赖包`predis/predis`

{{< highlight bash>}}
composer require predis/predis
{{< /highlight>}}

1. 重新生成自动加载文件

{{< highlight bash>}}
composer dumpautoload
{{< /highlight>}}

### 修改配置

#### 修改`.env`文件中的缓存驱动,会话驱动,redis连接设置

{{< highlight php>}}
<?php
#缓存驱动
#CACHE_DRIVER=array
CACHE_DRIVER=redis

#会话驱动
#SESSION_DRIVER=file
SESSION_DRIVER=redis

#redis连接驱动
REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379
{{< /highlight>}}

#### 配置session和cache使用不同的数据库
之所以这样做是避免清除缓存的时候，用户会话也一并清除

1. 修改`config/database.php`文件中的`redis`数组，复制一份`default`的配置，并依据需要修改其中的参数,这里最主要的两点是索引名称和`database`选项的修改

    {{< highlight php>}}
    <?php
    'session' => [
        'host' => env('REDIS_HOST'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 1,
    ],
    {{< /highlight>}}

1. 修改`config/session.php`文件中`connection`为`session`,这里的值，就是上一步设置的索引名称

    {{< highlight php>}}
    <?php
    'connection' => "session",
    {{< /highlight>}}

#### 重新加载配置

当配置完成以后，应该运行一下两条命令，重新加载配置，避免出现不必要的问题

1. 清除配置

{{< highlight bash>}}
php artisan config:clear
{{< /highlight>}}


1. 清空缓存

{{< highlight bash>}}
php artisan cache:clear
{{< /highlight>}}


#### 简单使用
Laravel提供`Cache`, `Redis`两种`Facade`来使用对redis进行操作,而这两种方式已存在一些区别

##### Redis　Facade
这种方式的使用通常是`Redis::redisCommandName`的形式，例如：
{{< highlight php>}}
<?php
Redis::set("hello", "world");
{{< /highlight>}}

当然`Redis::command("redisCommandName", params)`也是等价的

完整的命令列表，可以查看[redis commands](https://redis.io/commands),
在`\Illuminate\Redis\RedisManager`文件中，使用魔术方法调用redis的命令

{{< highlight php>}}
<?php
public function __call($method, $parameters)
{
    return $this->connection()->{$method}(...$parameters);
}
{{< /highlight>}}

##### Cache
Cache对不同的缓存进行部分封装，在使用这种方式的时候，需要制定所需要的存储
{{< highlight php>}}
<?php
Cache::store("redis)->put("hello", "world");
{{< /highlight>}}
具体的封装列表，可以查看``,如果对于需要的缓存的操作不复杂的话，建议直接使用这种方式

##### 注意事项
首先注意以下例子：
{{< highlight php>}}
<?php
Route::get("cache", function(){
    \Redis::setex("hello",10, "world");
    $res = \Redis::get("hello");
    $re = \Cache::store("redis")->get("hello");
    dd($res.'-'.$re);
});
{{< /highlight>}}

以上结果输入为

{{< highlight bash>}}
"world-"
{{< /highlight>}}

在redis-cli中，使用`KEYS *`进行查找结果如下，

{{< highlight bash>}}
127.0.0.1:6379> KEYS *
1) "hello"
{{< /highlight>}}

可见`hello`这个key已经存在，那么`Cache`的形式为什么取不到值呢，不急我们反过来试一下

{{< highlight php>}}
<?php
Route::get("cache", function(){
    // \Redis::setex("hello",10, "world");
    \Cache::store("redis")->put("hello", "world", 1);
    $res = \Redis::get("hello");
    $re = \Cache::store("redis")->get("hello");
    dd($res.'-'.$re);
});
{{< /highlight>}}

以上结果输入为

{{< highlight bash>}}
127.0.0.1:6379> KEYS *
1) "laravel_cache:hello"
{{< /highlight>}}

在redis-cli中，使用`KEYS *`进行查找可以发现，`hello`这个key并不存在，倒是有一个`laravel_cache:hello:`的前缀，
而且通过在redis-cli中获取到的值如下：

{{< highlight bash>}}
127.0.0.1:6379> get laravel_cache:hello
"s:5:\"world\";"
{{< /highlight>}}

显然是Cache对于其值进行了封装和转换，通过查找Cache相关文件，可以看到在`CacheManager.php`文件中，包含如下代码

{{< highlight php>}}
<?php
    /**
     * Get the cache prefix.
     *
     * @param  array  $config
     * @return string
     */
    protected function getPrefix(array $config)
    {
        return $config['prefix'] ?? $this->app['config']['cache.prefix'];
    }
{{< /highlight>}}

根据以上代码，我们可以追溯到`config/cache.php`文件中，包含如下配置：
{{< highlight php>}}
<?php
    'prefix' => env(
        'CACHE_PREFIX',
        str_slug(env('APP_NAME', 'laravel'), '_').'_cache'
    ),
{{< /highlight>}}

这就解释了为什么在Laravle中，使用两种不同的方式的时候，需要特别注意，使用哪一种方式存储，就是用哪一种方式读取




#### 参考文档
- [Laravel 下配置 Redis 让缓存、Session 各自使用不同的 Redis 数据库 ](https://laravel-china.org/topics/2466/laravel-configuration-under-the-redis-so-that-the-cache-session-each-use-a-different-redis-database)

- [Laradock](http://laradock.io/)