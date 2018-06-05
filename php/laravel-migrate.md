---
title: "Laravel 数据迁移浅析 使用篇"
date: 2018-06-05T11:53:19+08:00
draft: true
---


#### 命令查看

```bash
php artisan make:migration --help
```

#### 创建文件

在创建文件的时候,通过以下语句查看其创建选项
```bash
php artisan make:migration --help
```

对于创建迁移文件,有用的是以下几个选项

```bash
--create[=CREATE]  The table to be created.
--table[=TABLE]    The table to migrate.
--path[=PATH]      The location where the migration file should be created.
```

说明: 

- create: 创建一个新的数据表,后面指定表名,`--create=articles`

- table:  修改表字段,单独使用这个选项的时候,前提条件是数据表已经存在, `--table=articles`

- path: 指定一个相对于项目根目录的相对路径作为迁移文件的存放路径,前提条件是是该目录已经存在, `--path=database/migrations/temp`

##### 注意事项

1. 文件名称的一致性

    创建迁移文件的时候,需要保证文件名的唯一性,
    虽然在创建文件的时候,会自动添加当前日期以及微秒作为文件的前缀(如：`2018_06_01_014354_create_table_articles_table`), 
    
    但是需要注意的是,其所形成的迁移文件类名为
    `CreateTableArticlesTable`,尽管Laravel已经引入了类名检查的机制,但对于多人协作来说难免会存在创建相同类名的情况
    为了保证数据迁移时的唯一性，在`vendor/composer/autoload_classmap.php`文件的数组中,以类名作为key,以文件名称作为value

    为了迁移时能够正确执行，因此如果由于某些原因需要删除迁移文件或者分之合并以后,都需要执行`composer dumpautoload`,

    防止数据迁移的时候出现类名重复或者类文件不存在的情况

1. 文件结构

    在所创建的文件结构中,包含两个方法`up`和`down`,其中`up`方法执行的是迁移时需要进行的数据表操作,
    
    而对于`down`方法来说则是`up`操作的**逆操作**,如果没有对`down`方法进行维护,那么在执行迁移回滚的时候,
    虽然回滚成功(即`migrations`数据表的迁移记录回滚成功),
    
    但是`up`方法所进行的操作(比如增加,修改等)造成数据表结构的变更将不会进行回滚,造成重新迁移的时候,出现一系列的问题(比如:改名的时候,字段名称不存在, 增加的时候,字段名称已存在的问题)


#### 文件迁移

1. 迁移记录表结构

    迁移记录表的结构较为简单,总共也就两个字段
    - migration 记录迁移文件的文件名
    - batch 文件属于第几次迁移

    ```ansi
    mysql> select * from migrations;
    +------------------------------------------------+-------+
    | migration                                      | batch |
    +------------------------------------------------+-------+
    | 2014_10_12_000000_create_users_table           |     1 |
    | 2014_10_12_100000_create_password_resets_table |     1 |
    | 2018_06_01_014354_create_table_articles_table  |     2 |
    | 2018_06_01_014454_create_table_categories_table|     2 |
    | 2018_06_01_015356_create_tags_table            |     2 |
    +------------------------------------------------+-------+
    ```

1. 迁移命令
    
    使用`php artisan migrate --help`,查看迁移时的数据选项

    说明:
    - database, 指定需要连接的数据库

    - force, 在生产环境中强制执行数据迁移

    - path, 指定执行特定位置的迁移文件, 与创建迁移文件的`--path`选项相对应

    - pretend, 显示即将执行的sql语句,如果迁移文件中使用了`doctrine/dbal`提供的功能,将会提示错误,因为其操作必须依赖于数据表已存在的字段

    - seed 显示是否需要重新运行的迁移文件名称

    - step 将每一个文件作为一次迁移操作,也就是`migrations`表的`batch`会依据文件执行的顺序依次递增


1. 回滚上一次迁移

    ```bash
    php artisan migrate:rollback
    ```

    通过添加`--pretend`选项,可以查看数据即将执行的sql语句, 需要注意的是,这里是执行**上一次**操作,如果重复执行该命令,将会把迁移全部回滚,相当于执行全部执行回滚的命令

1. 回滚所有操作

    ```bash
    php artisan migrate:reset 
    ```
    通过添加相对应的选项可以查看不同的操作,具体选项与迁移时一样

1. 重建迁移
    
    ```bash
    php artisan migrate:refresh --seed
    ```
    执行该条语句,等价于先执行`migrate:reset`,然后再执行`migrate`


#### 参考资料

- [Laravel5.2 Database Migrations](https://laravel.com/docs/5.2/migrations)


