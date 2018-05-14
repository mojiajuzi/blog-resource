---
title: "MySQL中的约束"
date: 2018-05-14T23:28:26+08:00
draft: true
tags: ["Sql", "Mysql"]
---


#### NOT NULL 约束

NOT NULL约束主要是对一个数据表的列做出约束，表示该列的值不能为`NULL`

首先我们创建一个`articles`的数据表

```sql
create table articles(
    id INT NOT NULL AUTO_INCREMENT,
    title VARCHAR(50) NOT NULL,
    abstract VARCHAR(100),
    primary key(id)
    )ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

然后我们插入执行如下的语句进行插入数据
```bash
mysql> INSERT INTO articles(id, title, abstract) VALUES (NULL, "hello MySQL", "mysql abstract");
Query OK, 1 row affected (0.01 sec)
```

然后再执行一条语句
```
mysql> INSERT INTO articles(id, title, abstract) VALUES (NULL, "hello PHP", NULL);
Query OK, 1 row affected (0.00 sec)
```

接着我们再执行一条语句
```
mysql> INSERT INTO articles(id, title, abstract) VALUES (NULL, NULL, "nothing aabstract");
ERROR 1048 (23000): Column 'title' cannot be null

```

1. 对比第一条语句和第二条语句可知

    如果字段是`auto_increment`也就是自增长或者创建字段的时候没有显示声明`NOT NULL`的字段将可以插入`NULL`值

1. 对比第二和第三条语句可知

    如果显式的声明了`NOT NULL`,那么该字段值将不能插入`NULL`值

#### Unique(唯一约束）
Unique约束，也就是我们常说的唯一约束
首先我们对`articles`数据表插入一条与第一条一样的数据

```
mysql> INSERT INTO articles(id, title, abstract) VALUES (NULL, "hello MySQL", "mysql abstract");                                                  Query OK, 1 row affected (0.00 sec)
```
竟然写入成功了，我们通过查询来验证

```
mysql> select * from articles;
+----+-------------+----------------+
| id | title       | abstract       |
+----+-------------+----------------+
|  1 | hello MySQL | mysql abstract |
|  2 | hello PHP   | NULL           |
|  3 | hello MySQL | mysql abstract |
+----+-------------+----------------+
3 rows in set (0.00 sec)
```
我们看到，`articles`数据表中包含两条一模一样的数据，这是非常不合理的，接下来我们将会对数据表进行调整

1. 首先删除后面加入的一条记录
    ```
    mysql> delete from articles where id = 3;
    Query OK, 1 row affected (0.00 sec)
    ```

1. 修改`articles.title`字段，增加唯一性约束
    ```
    mysql> ALTER TABLE articles CHANGE title title  VARCHAR(50) NOT NULL UNIQUE;
    Query OK, 0 rows affected (0.02 sec)
    Records: 0  Duplicates: 0  Warnings: 0
    ```

1. 然后再尝试插入一条于第一条一样的数据
    ```
    mysql> INSERT INTO articles(id, title, abstract) VALUES (NULL, "hello MySQL", "mysql abstract");
    ERROR 1062 (23000): Duplicate entry 'hello MySQL' for key 'title'
    mysql> 
    ```
这样我们就可以保证在数据表`articles.title`的唯一性

#### Primary Key(主键约束)

主键约束于唯一约束的却别

1. 在一个数据表中，可以有多个唯一约束(unique)，但是有且只能有一个主键约束，

1. 唯一键可以为`NULL`但是主键不能


#### Foreign Key(外键约束)

一个表中的外键指向另外一个表的主键

1. 首先创建一个新表`categories`，用来保存文章的类型

    ```sql
    create table categories(
    id INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,
    primary key(id)
    )ENGINE=InnoDB DEFAULT CHARSET=utf8;
    ```

1. 然后我们在`articles`中增加一个`category_id`
    ```sql
     ALTER TABLE articles ADD COLUMN category_id INT NOT NULL;
    ```

1. 设置`category_id`作为`articles`的外键
    
    ```sql
    ALTER TABLE articles ADD CONSTRAINT fk_category_id FOREIGN KEY(category_id) REFERENCES categories(id);
    ```

1. 接着创建两条数据在`categories`中
    ```
    mysql> INSERT INTO categories VALUES(null, 'front-end'),(NULL, 'back_end');
    Query OK, 2 rows affected (0.01 sec)
    Records: 2  Duplicates: 0  Warnings: 0
    ```

假设我们先插入一条数据到`articles`中，指定一个不是`categories`主键id的值作为`articles.category_id`的值

```
mysql> INSERT INTO articles VALUES(NULL, 'Js', NULL,3);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`myconstraint`.`articles`, CONSTRAINT `fk_category_id` FOREIGN KEY (`category_id`) REFERENCES `categories` (`id`))
```

如果修改一下值，如下所示
```
mysql> INSERT INTO articles VALUES(NULL, 'Js', NULL,1);
Query OK, 1 row affected (0.00 sec)
```

由此可见，使用外键约束，可以保证数据的一致性

#### ENUM (枚举约束)
通过枚举约束，可以限定表中列的值的范围

1. 修改`categories`表，增加一列`status`用来标示，该记录是否激活(`activ`, `inactive`)
    ```sql
    ALTER TABLE categories ADD COLUMN status ENUM('active','inactive') DEFAULT 'active';
    ```

1. 然后尝试更新`status`的值为非范围内的值

    ```
    mysql> update categories set status='hello' where id = 1;
    ERROR 1265 (01000): Data truncated for column 'status' at row 1
    ```

#### SET
一个SET列可以包含零个或多个值，但是每一个值都必须在`SET`指定的范围内

1. 首先创建一个学生表
    ```
    mysql> CREATE TABLE Students(Id INTEGER, Name VARCHAR(55), 
    -> Certificates SET('A1', 'A2', 'B1', 'C1')); 
    ````

1. 然后执行以下操作
    ```
    mysql> INSERT INTO Students VALUES(1, 'Paul', 'A1,B1');
    mysql> INSERT INTO Students VALUES(2, 'Jane', 'A1,B1,A2');
    mysql> INSERT INTO Students VALUES(3, 'Mark', 'A1,A2,D1,D2');
    ```

1. 其结果如下
    ```
    mysql> SELECT * FROM Students;
    +------+------+--------------+
    | Id   | Name | Certificates |
    +------+------+--------------+
    |    1 | Paul | A1,B1        |
    |    2 | Jane | A1,A2,B1     |
    |    3 | Mark | A1,A2        |
    +------+------+--------------+
    ```


    #### 参考资料

    1. [Previous Next MySQL constraints](http://zetcode.com/databases/mysqltutorial/constraints/)

    1. [mysql alter 语句用法,添加、修改、删除字段等](https://blog.csdn.net/wyswlp/article/details/8881103)