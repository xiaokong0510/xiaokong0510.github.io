---
title: Redis 五大数据结构及常用指令
categories:
  - 数据库
  - Redis 
tags:
  - Redis
date: 2021-06-20
description: 汇总一下 Redis 五大数据结构常用指令。
---

官网：https://redis.io/        http://www.redis.cn/

Redis 命令参考：  http://doc.redisfans.com/ ， http://www.redis.cn/commands.html

Redis（Remote Dictionary Server )，即远程字典服务，是一个使用 C 语言开发的数据库。

**Redis 的数据是存在内存中的**

**Redis 除了做缓存之外，也经常用来做分布式锁，甚至是消息队列。**

**Redis 提供了多种数据类型来支持不同的业务场景。Redis 还支持事务 、持久化、Lua 脚本、多种集群方案。**

## 常见数据结构

- string
- list
- hash
- set
- sorted set

## Redis 命令

### Key（键）
1. **检查/ 删除 /移动 key：**

`EXISTS key`   ：若 key 存在，返回 1 ，否则返回 0 ；
`DEL key [key ...]`  ： 删除给定的一个或多个 key，不存在的 key 会被忽略，返回 被删除 key 的数量；
`MOVE key db`  ：移动到给定的数据库 db 当中，成功返回 1 ，失败则返回 0 。如果源数据库和目标数据库有相同名字的给定 key ，或者 key 不存在于当前数据库，那么 MOVE 没有任何效果
`TYPE key`  ： 返回 key 所储存的值的类型
`RENAME key newkey`  ： 将 key 改名为 newkey；当  key 不存在时，返回一个错误；当 newkey 已经存在时， RENAME 命令将覆盖旧值



2. **生存时间：**

`TTL key`  ：  TTL, time to live，以秒为单位
`PTTL key` ：以毫秒为单位
- 当 key 不存在时，返回 -2 ；
- 当 key 存在但没有设置剩余生存时间时，返回 -1 ；
- 否则，以秒为单位，返回 key 的剩余生存时间

`EXPIRE key seconds`  ： 设置生存时间，单位为 s，设置成功返回 1，
`PEXPIRE key milliseconds`  ： 设置生存时间，单位为 ms

`EXPIREAT key timestamp`  ： 设置生存时间，参数为 UNIX 时间戳
`PEXPIREAT key milliseconds-timestamp`  ： 设置生存时间，以毫秒为单位的 UNIX 时间戳

`PERSIST key timestamp`  ： 移除给定 key 的生存时间，即设置为永久；当生存时间移除成功时，返回 1 ；如果 key 不存在或 key 没有设置生存时间，返回 0 



3. **查找/扫描：**

`KEYS pattern`  ：查找所有符合给定模式 pattern 的 key 

- KEYS * 匹配数据库中所有 key 。
- KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。
- KEYS h*llo 匹配 hllo 和 heeeeello 等。
- KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo 

`SCAN cursor [MATCH pattern] [COUNT count]`  ：查找所有符合给定模式 pattern 的 key 

### String（字符串）

1. **基本操作：**

`SET key value [EX seconds] [PX milliseconds] [NX|XX]`  ： 设置 key-value 类型的值；key 已经存在则覆写旧值，无视类型，这个键原有的 TTL 也将被清除。支持以下参数

- EX second ：设置键的过期时间为 second 秒。 SET key value EX second 效果等同于 SETEX key second value 
- PX millisecond ：设置键的过期时间为 millisecond 毫秒。 SET key value PX millisecond 效果等同于 PSETEX key millisecond value 
- NX ：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value 。
- XX ：只在键已经存在时，才对键进行设置操作。



`GET key`  ： 根据 key 获得对于的 value，如果 key 不存在那么返回特殊值 nil 

`STRLEN key`  ： 返回 key 所储存的字符串值的长度 
`SETRANGE key offset value`  ： 从偏移量 offset 开始设置 key 所保存的 value，返回被 SETRANGE 修改之后字符串的长度；不存在的 key 当作空白字符串处理，空白处被"\x00"填充

`GETRANGE key start end`  ：返回截取后的字符串， 包括 start 和 end 在内；负数偏移量表示从字符串最后开始计数，-1 表示最后一个字符， -2 表示倒数第二个

`SETEX key seconds value`  ： 将值 value 关联到 key ，并将 key 的生存时间设为 seconds (以秒为单位)

`SETNX key value`  ：即 【SET if Not eXists】。 当且仅当 key 不存在时，将 key 的值设为 value ；若给定的 key 已经存在，则不做任何动作。设置成功，返回 1 ；设置失败，返回 0 。



2. **批量设置：**

`MSET key value [key value ...]`  ： 同时设置一个或多个 key-value 对；是一个原子性(atomic)操作，所有给定 key 都会在同一时间内被设置
`MGET key [key ...]`  ：返回所有(一个或多个)给定 key 的值。



3. **计数器：**

`DECR key`  ： 将 key 中储存的数字值增一；如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作
`INCRBY key`  ： 将 key 所储存的值加上增量 increment 。
`INCR key`  ： 将 key 中储存的数字值减一；如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECR 操作
`INCRBY key`  ： 将 key 所储存的值减去减量 decrement 

### List（列表）

`RPUSH key value [value ...]`  ： 右边插入；返回执行 RPUSH 操作后，表的长度
`LPOP key`  ： 移除并返回列表 key 最左边的元素
`LPUSH key value [value ...]`  ： 左边插入；返回执行 LRPUSH 操作后，表的长度
`RPOP key`  ： 移除并返回列表 key 最右边的元素


![redis-list](https://image.kongxiao.top/20210922175322.png)



`LLEN key`  ： 返回列表 key 的长度，如果 key 不存在，则 key 被解释为一个空列表，返回 0，
如果 key 不是列表类型，返回一个错误
`LRANGE key start stop`  ： 返回列表 key 中指定区间内的元素，-1表示倒数第一
`LINDEX key index`  ： 返回列表 key 中，下标为 index 的元素

`LINSERT key BEFORE|AFTER pivot value`  ： 将值 value 插入到列表 key 当中，位于值 pivot 之前或之后。

- 当 pivot 不存在于列表 key 时，不执行任何操作；
- 当 key 不存在时， key 被视为空列表，不执行任何操作；

- 如果 key 不是列表类型，返回一个错误；
- 如果命令执行成功，返回插入操作完成之后，列表的长度；

- 如果没有找到 pivot ，返回 -1 ；
- 如果 key 不存在或为空列表，返回 0



`LREM key count value`：根据参数 count 的值，移除列表中与参数 value 相等的元素。返回被移除元素的数量

- count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count ；
- count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值；

- count = 0 : 移除表中所有与 value 相等的值


### Hash（哈希表）

`HSET key filed value [filed value ...]`：设置 key 中的域 field 的值设为 value
`HGET key filed`：返回指定 key 中给定域 field 的值

`HLEN key`：返回指定 key 中域的数量

`HGETALL key`：返回指定key 所有的域和值

`HEXISTS key filed`：查看指定key，指定域 field 是否存在

`HDEL key filed[field ...]`：删除指定 key 中，一个或多个给定的 field 

`HKEYS key`：返回指定key 中，所有的 field

`HVALS key`：返回指定key，所有的 field 的 value

`HSETNX key field value`：当且仅当域 field 不存在，将指定 key 中的域 field 的值设置为 value 

`HINCRBY key field increment`：为指定 key 中的域 field 的值加上增量 increment 


### Set （集合）

`SADD key member [member ...]` ：添加元素

`SMEMBERS key`：查看所有元素
`SISMEMBERS key member`：判断 member 元素是否是集合 key 的成员

`SCARD key`：查看元素个数

`SREM key member [member ...]`：移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略

`SPOP key`：移除并返回集合中的一个随机元素

`SRANDMEMBER key [count]`：返回集合中的一个随机元素

- 如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素**各不相同**。如果 count 大于等于集合基数，那么返回整个集合
- 如果 count 为负数，那么命令返回一个数组，数组中的元素**可能会重复出现多次**，而数组的长度为 count 的绝对值

`SMOVE source destination member`：将 member 元素从 source 集合移动到 destination 集合

**数学集合类：**

`SDIFF key [key ...]`：返回所有给定集合之间的差集
`SINNER key [key ...]`：返回所有给定集合之间的交集
`SUNION key [key ...]`：返回所有给定集合之间的并集

`SDIFFSTORE destination  key [key ...]`：返回所有给定集合之间的差集，将结果保存到 destination 集合
`SINNERSTORE destination key [key ...]`：返回所有给定集合之间的交集，将结果保存到 destination 集合
`SUNIONSTORE destination key [key ...]`：返回所有给定集合之间的并集，将结果保存到 destination 集合


### ZSet（Sorted Set 有序集合）

在 SET 基础上，加上一个 scroe 值

`ZADD key score member [[score member] [score member] ...]`：将一个或多个 member 元素及其 score 值加入到有序集 key 当中

`ZRANGE key start stop [WITHSCORES]`：返回有序集 key 中，指定区间内的成员，其中成员的位置按 score 值递增(从小到大)来排序

`ZREVRANGE key start stop [WITHSCORES]`：同上，逆序

`ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`：返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列

`ZREVRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`：同上，逆序

`ZSCORE key member`：返回有序集 key 中，成员 member 的 score 值

`ZRANK key member`：返回有序集 key 中成员 member 的排名

`ZCARD key`：查看元素个数

`ZCOUNT key min max`：返回有序集 key 中， score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max )的成员的数量

`ZREM key member [member ...]`：移除有序集 key 中的一个或多个成员

