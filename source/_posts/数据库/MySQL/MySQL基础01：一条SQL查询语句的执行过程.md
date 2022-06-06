---
title: MySQL基础01：一条SQL查询语句的执行过程
categories:
  - 数据库
  - MySQL
tags:
  - MySQL
date: 2021-06-20
description: MySQL 入门课程，基础架构部分，SQL 查询语句的执行过程。买的课，迟早要学完。
---

本文是学习 MySQL 的过程中的笔记记录整理。

学习资料：  [极客时间 MySQL实战45讲-丁奇](https://time.geekbang.org/column/intro/100020801)

内容包括：

1. MySQL 数据库的基础架构：
2. SQL 查询语句的执行过程；
3. 连接器、分析器、优化器、执行器、存储引擎等的作用

## 1 MySQL 基础架构

经典的 MySQL 的基本架构示意图：

![img](https://image.kongxiao.top/20210806172606.png)

MySQL 可以分为 **Server 层** 和 **存储引擎层** 两部分。

- **Server 层** ：包括连接器、查询缓存、分析器、优化器、执行器等，涵盖 MySQL 的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

- **存储引擎层** ： 负责数据的存储和提取。其架构模式是插件式的，支持 InnoDB、MyISAM、Memory 等多个存储引擎。现在最常用的存储引擎是 InnoDB (MySQL 5.5.5 版本开始成为了默认存储引擎) 。

**不同的存储引擎共用一个 Server 层**

建表语句示例：使用 `ENGINE=InnoDB` 来指定存储引擎，不指定引擎类型，默认使用的就是 InnoDB

```sql
CREATE TABLE `user_info` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
```


## 2 连接器

连接器 负责跟客户端建立连接、获取权限、维持和管理连接。

```bash
# 启动MySQLl服务
net start mysql
# 连接MySQL
mysql -h$ip -P$port -u$user -p
```

如果用户名或密码不对，会报错`"Access denied for user"`，然后客户端程序结束执行。

如果用户名密码认证通过，连接器会到权限表里面查出账户所拥有的权限。之后，这个连接里面的权限判断逻辑，都将依赖于此时读到的权限。

也就是说 **连接建立以后，权限就确定下来；如果发生变化，需要下次重新连接时生效。**

```sql
# 查看连接情况
show processlist
```

![image-20210807092604463](https://image.kongxiao.top/20210807092605.png)

- 客户端如果长时间无动作，连接器会将其断开。这个时间是由参数 `wait_timeout` 控制的，默认值是 8 小时，可通过以下指令查看

```sql
show variables like 'wait_timeout'
```

![image-20210807092717997](https://image.kongxiao.top/20210807092719.png)

- 如果在连接被断开之后，客户端再次发送请求的话，就会收到一个错误提醒：`Lost connection to MySQL server during query`，就需要重连

**长连接与短连接：**

- 长连接：连接成功后，如果客户端持续有请求，则一直使用同一个连接，（为了提升数据库并发性，可以建立一个数据库连接池）；长连接如果长期闲置，MySQL 会 8 小时后（默认时间）主动断开该连接；

- 短连接：每次执行完很少的几次查询就断开连接，下次查询再重新建立一个

但是长时间使用长连接占用内存涨得会特别快， 因为 MySQL 在执行过程中临时使用的内存是管理在连接对象里面的，这些资源会在连接断开的时候才释放。所以如果长连接累积下来，可能导致内存占用太大，被系统强行杀掉（OOM），从现象看就是 MySQL 异常重启了。

解决方案：

1. 定期断开长连接。使用一段时间，或者程序里面判断执行过一个占用内存的大查询后，断开连接，之后要查询再重连。
2. MySQL 5.7 或更新版本，可以在每次执行一个比较大的操作后，通过执行 mysql_reset_connection 来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。

可以类比 HTTP 的长连接和短连接：  [[HTTP长连接、短连接究竟是什么？]](https://www.cnblogs.com/gotodsp/p/6366163.html)



查看mysql最大连接数：

```sql
show variables like 'max_connections'
```

## 3 查询缓存

建立连接完成后，可以开始执行 select 语句，来到第二步：查询缓存

MySQL 执行过的查询语句，会以 key-value 对的形式，被直接缓存在内存中，key 是查询的语句，value 是查询的结果。

在执行一个查询请求时，会先查询缓，如果能够直接在这个缓存中命中 key，那么就直接返回 value；否则才继续后面的执行阶段。

但是缓存带来了额外的开销，每次查询后都要做一次缓存操作，失效后还要销毁。**只要有对一个表的更新，这个表上所有的查询缓存都会被清空**， 对于更新压力大的数据库来说，查询缓存的命中率会非常低。

MySQL 提供了"按需使用"的方式。 SQL_CACHE 显式指定，像下面这个语句一样：设置缓存的两个参数：

- `query_cache_type` 

- `query_cache_size`

**还可以通过 sql_cache 和 sql_no_cache 来控制某个查询语句是否需要缓存**

```sql
select sql_no_cache count(*) from usr;
select sql_cache count(*) from usr;
```



MySQL 8.0 版本后移除，因为这个功能不太实用！！

## 4 分析器

```sql
select * from T where id=10；
```



分析器，包括 **词法分析** 和 **语法分析**

- 词法分析：识别出里面的字符串分别是什么，代表什么。MySQL 从输入的 "select" 这个关键字识别出来，这是一个查询语句。把字符串 “T” 识别成“表名 T”，把字符串 “id” 识别成 “列 id” 。
- 语法分析：根据词法分析的结果，语法分析器会根据语法规则，判断输入的这个 SQL 语句是否满足 MySQL 语法。如果不对会报 `“You have an error in your SQL syntax”`

## 5 优化器

一条 SQL 语句可能有不同的执行逻辑（或者顺执行顺序），而优化器就是选择最优的执行顺序。

在表里面有多个索引的时候，决定使用哪个索引；或者在一个语句有多表关联（join）的时候，决定各个表的连接顺序。

但是 优化器判断的有的时候未必是正确的！后续再详细学习下

可以使用 explain 指令来查看 SQL 执行计划。

## 6 执行器

MySQL 通过分析器知道了要做什么，通过优化器知道了该怎么做，于是就进入了执行器阶段，开始执行语句

**开始执行的时候，要先判断一下对这个表 T 有没有执行查询的权限**，如果没有，就会返回没有权限的错误。 (在工程实现上，如果命中查询缓存，会在查询缓存返回结果的时候，做权限验证。查询也会在优化器之前调用 precheck 验证权限)。

```sql
mysql> select * from T where ID=10;

ERROR 1142 (42000): SELECT command denied to user 'b'@'localhost' for table 'T'
```

如果有权限，就打开表继续执行。打开表的时候，执行器就会根据表的引擎定义，去使用这个引擎提供的接口。

**为什么对权限的检查不在优化器之前做？**因为有些时候，SQL 语句要操作的表不只是 SQL字面上那些。比如有个触发器，得在执行器阶段（过程中）才能确定，优化器阶段前是无能为力的



