# Memcached的简易使用
![Memcached简介](http://upload.ouliu.net/i/20180310211106206n0.jpeg)

## Memcached简介
Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的hashmap。其守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过Memcached协议与守护进程通信。

## 常用的命令

| 命令 | 说明 | 用法 |
| :---: | :---: | :---: |
| set | 设置key的值value |  set key flags exptime bytes [noreply] value |
| add | 添加key的值value（不覆盖原值） |  add key flags exptime bytes [noreply] value |
| replace | 替换原来的值 | replace key flags exptime bytes [noreply] value |
| append | 原来的值后面追加值 | append key flags exptime bytes [noreply] value |
| prepend | 原来的值前面追加值 | prepend key flags exptime bytes [noreply] value |
| get | 获取存储在key中的value | get key [...key1] |
| delete | 删除已存在的key | delete key [noreply] |
| incr/decr | 对已存在的key的值进行自增/自减 | incr/decr key increment_value |

## 使用场景
* 对频繁获取的数据进行缓存(减轻DB压力）；
* 用锁的机制控制流量；
* 多服务器间共享数据（比如session）

## 备注
Redis（SSDB）数据类型丰富，Memcached数据类型单一，Memcached是内存式缓存系统，Redis是更像是内存式数据库。两者干嘛要比较呢？？？？个人感觉Memcached+SSDB就可以支撑很多项目了，如果实在是那种实时要求，并发大，那可以考虑Redis。

## 参考
1.[https://www.w3cschool.cn/memcached/](https://www.w3cschool.cn/memcached/)