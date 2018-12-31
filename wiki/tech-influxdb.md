# InfluxDB的简单使用

## 关于时序数据库

#### 1. Time series

```
A time series is a series of data points indexed (or listed or graphed) in time order. Most commonly, a time series is a sequence taken at successive equally spaced points in time.
```
时序（时间序列）指基于时间顺序的被索引化的数据点的集合。一般而言，时间序列指的是在时间上连续等距的点的序列(一串按时间维度索引的数据)。

####  2. Time series database

```
A time series database (TSDB) is a software system that is optimized for handling time series data, arrays of numbers indexed by time (a datetime or a datetime range). In some fields these time series are called profiles, curves, or traces.
```

#### 3. 时序数据库的特点

  * 压缩能力
  * 高性能写入能力
  * 多维度查询能力（索引）
  * 分析（聚合）能力
  * ...

## InfluxDB简介

```
InfluxDB is a time series database designed to handle high write and query loads. InfluxDB is meant to be used as a backing store for any use case involving large amounts of timestamped data, including DevOps monitoring, application metrics, IoT sensor data, and real-time analytics.
```

大致的意思就是InfluxDB是一个时序数据库，它设计的初衷是为了解决高性能写入和查询负载。如果一个实例具有大量的包含时间戳的数据，InfluxDB很适合用作它的存储，比如DevOps监控数据、应用的指标数据、物联网传感器数据、实数分析的数据。

## 基本概念

#### 1. 安装。。（省略，手册有）

使用方式：
* 客户端命令行
* HTTP API接口
* 各种语言API包

#### 2. 配置。。（省略，手册有）

#### 3. 基本概念

#####  3. 1. 名词对比

|influxDB|传统数据库|
|:---:|:---:|
|database|database|
|measurement|table|
|point|raw|

#####  3. 2. InfluxDB中的point(就是db中的一行数据，由三部分组成)

point由时间戳（time）、数据（fields）、标签（tags）组成。(插入语句有体现该特征)

|point|释义|
|:---:|:---:|
|time|每个数据记录时间，是数据库中的主索引(会自动生成)|
|fields|各种记录值（没有索引的属性）也就是记录的值|
|tags|各种有索引的属性|

#####  3. 3. 基本命令

#####  3. 3. 1. 登录登出

```
influx # 登录
exit # 登出
```

#####  3. 3. 2. 数据库操作

```
CREATE DATABASE db_name # 创建数据库
SHOW DATABASES # show数据库
USE db_name # 选择数据
DROP DATABASE db_name # 删除数据库
SHOW MEASUREMENTS # 显示该数据库中的表
#插入数据时指定表名时会自动创建表
DROP MEASUREMENT "measurementName" # 删除表
```

#####  3. 3. 3. 增（建议看文档理解一下insert语句个字段含义）

```
INSERT cpu,host=serverA,region=us_west value=0.64
```
释义：time为默认字段，主索引(对应point的time部分)，自动生成；表名：cpu；字段：host、region、alue、time；索引（对应point的tags部分）：time、host、region；值（对应point的fields部分）：value

#####  3.  3. 4. 保存策略(RETENTION POLICIES，当数据超过了指定的时间之后，就会被删除)

```
SHOW RETENTION POLICIES ON "db_name" # 查看当前数据库的保存策略
CREATE RETENTION POLICY "rp_name" ON "db_name" DURATION 30d REPLICATION 1 DEFAULT # 创建新的保存策略
ALTER RETENTION POLICY "rp_name" ON db_name" DURATION 3w DEFAULT # 修改
DROP RETENTION POLICY "rp_name" ON "db_name" # 删除
```

释义：rp_name：策略名，db_name：数据库名，30d：保存30天，h（小时），w（星期），REPLICATION 1：副本个数，DEFAULT 设为默认的策略

#####  3. 3. 5. 查询

```
SELECT "host", "region", "value" FROM "cpu" # 普通查询
```

#####  3. 3. 6. 连续查询(CONTINUOUS QUERIES，将数据定时归档)

* 当数据超过保存策略里指定的时间之后，就会被删除。如果不想完全删除掉，比如做一个数据统计采样，可以把原先每秒的数据，存为每小时的数据，让数据占用的空间大大减少（以降低精度为代价）。

* 定时归档数据，减少空间

```
SHOW CONTINUOUS QUERIES # 查看当前策略
CREATE CONTINUOUS QUERY cq_30m ON db_name BEGIN SELECT mean(temperature) INTO weather30m FROM weather GROUP BY time(30m) END
SHOW MEASUREMENTS # 创建新策略（摘自手册）
DROP CONTINUOUS QUERY <cq_name> ON <db_name> # 删除策略
```

释义：cq_30m：连续查询的名字，db_name：具体的数据库名，mean(temperature): 算平均温度，weather：当前表名，weather30m：存新数据的表名，30m：时间间隔为30分钟

#####  3. 3. 7. 用户管理(详情见手册)

#####  3. 3. 8. 函数(重点！)

```
COUNT()  DISTINCT()  MEAN()  MEDIAN() SPREAD() SUM()... # 聚合函数
TOP() BOTTOM() FIRST() LAST() MAX() MIN() PERCENTILE() ...# 选择函数
DERIVATIVE() DIFFERENCE() ELAPSED() MOVING_AVERAGE() NON_NEGATIVE_DERIVATIVE() STDDEV() ...# 变换函数
GROUP BY time(n) # 分类
```
备注：这里不展开讲每个函数的作用，可以查看相关资料和手册对以上函数进行试验！

## TSM存储引擎、TimeSeries Index

xxxx

## 性能报告

参考5，6

## 总结

> 使用场景：DevOps监控数据、应用的指标数据、物联网传感器数据、实数分析的数据...

> 备注：时序数据库属于细分领域的结果，这类产品其实有很多的选择，合适即可，如果有兴趣对其进行深究的话，先学好go.

> 有空可以整理一下市面上的主流的数据库。

## 参考

1. [https://en.wikipedia.org/wiki/Time_series](https://en.wikipedia.org/wiki/Time_series)
2. [https://docs.influxdata.com/influxdb/v1.7/](https://docs.influxdata.com/influxdb/v1.7/)
3. [http://hbasefly.com/2017/12/08/influxdb-1/](http://hbasefly.com/2017/12/08/influxdb-1/)
4. [https://www.bookstack.cn/read/influxdb-handbook](https://www.bookstack.cn/read/influxdb-handbook)
5. [https://zhuanlan.zhihu.com/p/42287416](https://zhuanlan.zhihu.com/p/42287416)
6. [https://zhuanlan.zhihu.com/p/32710333](https://zhuanlan.zhihu.com/p/32710333)