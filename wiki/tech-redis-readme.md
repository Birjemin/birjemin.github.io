# Redis的简易使用

## Redist简介
![git简介](http://upload.ouliu.net/i/20180310151124il1mb.jpeg)

```
REmote DIctionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。
Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
``` 

## 特点
* 支持持久化
* 数据结构类型丰富
* 支付master-slave模式的数据备份
* 原子性

## 数据类型
* string类型，一个key对应一个value（键值对）。一个键最大能存储512MB。

| 方法 | 用途 | 示例 |
| :---: | :---: | :---: |
| GET | 获取值 | GET name |
| SET | 设置值 | SET name val |
| DEL | 删除值 | DEL name |
| INCR | 自增1 | INCR key |
| DECR | 自减1 | DECR key |
| INCRBY | 自增整数amount | INCRBY key amount |
| DECRBY | 自减整数amount | DECRBY key amount |
| INCRBYFLOAT | 自增浮点值amount | INCRBYFLOAT key amount |

还有一些请看下面的链接，比如批量设置、设置过期时间。
[更多使用方式](http://www.runoob.com/redis/redis-strings.html)

* hash类型，是一个string类型的field和value的映射表，hash特别适合用于存储对象。每个hash可以存储2^32 - 1键值对（40多亿）。

| 方法 | 用途 | 示例 |
| :---: | :---: | :---: |
| HGET | 获取值 | HGET hash-key sub-key1 |
| HSET | 设置值 | HSET hash-key sub-key1 value1 |
| HGETALL | 获取散列所有的值 | HGETALL hash-key |
| HDEL | 删除值 | HDEL hash-key sub-key1 |
| HMSET | 设置散列中的一个或多个键的值 | HMSET hash-key sub-key1 value1[sub-key value...] |
| HMGET | 获取散列中的一个或多个键的值 | HMGET hash-key sub-key1 [sub-key...] |
| HLEN | 返回散列中的键值对数量 | HLEN hash-key |
| HKEYS | 获取散列中的所有键 | HKEYS hash-key |

[更多使用方式](http://www.runoob.com/redis/redis-hashes.html)

* list类型，是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。一个列表最多可以包含 232 - 1 个元素 (每个列表超过40亿个元素)。

| 方法 | 用途 | 示例 |
| :---: | :---: | :---: |
| RPUSH | 将给定值推入到列表右端 | RPUSH key name |
| LPUSH | 将给定值推入到列表左端 | LPUSH key name |
| RPOP | 从列表右端抛出一个值，并且返回其值 | RPOP key |
| LPOP | 从列表左端抛出一个值，并且返回其值 | LPOP key |

[更多使用方式](http://www.runoob.com/redis/redis-lists.html)

* set类型，是String类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。集合中最大的成员数为 2^32 - 1 (每个集合可存储40多亿个成员)。

| 方法 | 用途 | 示例 |
| :---: | :---: | :---: |
| SADD | 往集合添加元素 | SADD key item |
| SREM | 从集合移除元素 | SREM key item |
| SMEMBERS | 返回集合中的所有成员 | SMEMBERS key |

[更多使用方式](http://www.runoob.com/redis/redis-sets.html)

* zset类型，有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关	联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。有序集合的成员是唯一的,但分数(score)却可以重复。集合中最大的成员数为 2^32 - 1 (每个集合可存储40多亿个成员)。

| 方法 | 用途 | 示例 |
| :---: | :---: | :---: |
| ZADD | 往集合添加元素 | ZADD key score item |
| ZREM | 从集合移除元素 | ZREM key item |

## 使用场景

| 类型 | 用途示例 |
| :---: | :---: |
| string | 计数器，数据缓存 |
| hash | 存储用户信息，数据缓存 |
| list | 入队消费，抢购排队 |
| set | 可以求交集，差集，比如微博共同好友 |
| zset | 可以排序，比如全班同学成绩 |

更多的场景:

> [https://www.cnblogs.com/NiceCui/p/7794659.html](https://www.cnblogs.com/NiceCui/p/7794659.html)

> [https://www.cnblogs.com/xiaoxi/p/7007695.html](https://www.cnblogs.com/xiaoxi/p/7007695.html)

> [https://www.cnblogs.com/mrhgw/p/6278619.html](https://www.cnblogs.com/mrhgw/p/6278619.htmls)

> [http://blog.csdn.net/truong/article/details/39228731](http://blog.csdn.net/truong/article/details/39228731)

## 补充
redis还有事务、发布订阅等功能。

## 参考
1. [http://blog.csdn.net/xlgen157387/article/details/60958657](http://blog.csdn.net/xlgen157387/article/details/60958657)
2. [http://www.runoob.com/redis/redis-tutorial.html](http://www.runoob.com/redis/redis-tutorial.html)
3. [http://doc.redisfans.com/](http://doc.redisfans.com/)
4. [https://www.cnblogs.com/mrhgw/p/6278619.html](https://www.cnblogs.com/mrhgw/p/6278619.html)
5. [http://blog.csdn.net/yoko_luo/article/details/52303867](http://blog.csdn.net/yoko_luo/article/details/52303867)