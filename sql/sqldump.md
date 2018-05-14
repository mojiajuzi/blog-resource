---
title: "mysqldump备份工具的简单使用"
date: 2018-01-31T20:40:58+08:00
draft: true
description: 在Cli中使用命令行备份数据 
tags: ["Sql", "Mysql"]
---


1. 导出某个数据库中的所有数据表,不包含表中的数据
```mysql
    mysqldump -hlocalhost -utest -p -P3306 -d database_name > /path
```

1. 导出数据库中某张表的结构,不包含表中的数据
```mysql
    mysqldump -hlocalhost -utest -p -P3306 -d database_name table_name > /path
```

1. 导出数据库中的所有数据表及表中所包含的数据
```mysql
    mysqldump -hlocalhost -utest -p -P13306 database_name > /path
```

1. 导出数据表的结构以及所有的数据
```mysql
    mysqldump -hlocalhost -utest -p -P13306 database_name table_name > /path
```

1. 导出数据表结构以及部分数据
```
    mysqldump -hlocalhost -utest -p -P13306 database_name table --where="where_condition" > /path
```