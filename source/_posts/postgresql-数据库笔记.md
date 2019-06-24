---
title: postgresql 数据库笔记
date: 2017-03-18 23:06:50
tags:
    - PostgreSQL
---
+  最近学习 PostgreSQL 的笔记。（不定时更新）
 <!-- more -->
## 学习环境是 Mac 系统

安装最后自动初始化了一个数据库路径为 /usr/local/val/postgres

启动Postgresql
```php
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
```
停止Postgresql
```php
pg_ctl -D /usr/local/var/postgres stop -s -m fast
```
新建一个数据库用户
```php
createuser weixinadmin -P
```

为这个用户新建一个数据库
```php
createdb database -O weixinadmin -E UTF8 -e
```

用这个用户访问数据库
```php
psql -U weixinadmin -d weixindb -h 127.0.0.1

```
PostgreSQL 数据库操作
显示已创建的数据库：
```php
\l  
```

在不连接进 PostgreSQL 数据库的情况下，也可以在终端上查看显示已创建的列表：
```php
psql -l
```

连接数据库
```php
\c dbname
```

显示数据库表
```php
\d 
```

创建一个名为 test 的表
```php
CREATE TABLE test(id int, text VARCHAR(50));
```

插入一条记录
```php
INSERT INTO test(id, text) VALUES(1, 'sdfsfsfsdfsdfdf');
```

查询记录
```php
SELECT * FROM test WHERE id = 1;
```

更新记录
```php
UPDATE test SET text = 'aaaaaaaaaaaaa' WHERE id = 1;
```

删除指定的记录
```php
DELETE FROM test WHERE id = 1;
```

删除表
```php
DROP TABLE test;
```

删除数据库
```php 
DROP DATABASE dbname;
```
或者利用 dropdb 指令，在终端上删除数据库
```php
dropdb -U user dbname
```

