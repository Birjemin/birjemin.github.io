# MongoDB的使用
![MongoDB](http://upload.ouliu.net/i/20180310221445e12ar.png)

## 简介
MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

## NoSQL特点
* 代表着不仅仅是SQL(Not Only SQL)
* 键值对存储，列存储，文档存储，图形数据库
* 最终一致性，而非ACID属性
* 非结构化和不可预知的数据
* CAP定理(!!)
* 高性能，高可用性和可伸缩性

## MongoDB概念
1. SQL术语对比
| SQL术语/概念 | MongoDB术语/概念 |	解释/说明 |
| :---: | :---: | :---: |
| database | database |	数据库 |
| table	| collection | 数据库表/集合 |
| row | document | 数据记录行/文档 |
| column | field | 数据字段/域 |
| index | index | 索引 |
| table joins | 	| 表连接,MongoDB不支持 |
| primary key |	primary key | 主键,MongoDB自动将_id字段设置为主键 |

2. SQL条件对比

| 操作 | 格式 | 范例 | RDBMS中的类似语句 |
| :---: | :---: | :---: | :---: |
| 等于 | {<key>:<value>} | db.col.find({"by":"菜鸟教程"}).pretty() | where by = '菜鸟教程' |
| 小于 | {<key>:{$lt:<value>}} | db.col.find({"likes":{$lt:50}}).pretty() | where likes < 50 |
| 小于或等于 | {<key>:{$lte:<value>}} | db.col.find({"likes":{$lte:50}}).pretty() | where likes <= 50 |
| 大于 | {<key>:{$gt:<value>}} | db.col.find({"likes":{$gt:50}}).pretty() | where likes > 50 |
| 大于或等于 | {<key>:{$gte:<value>}} | db.col.find({"likes":{$gte:50}}).pretty() | where likes >= 50 |
| 不等于 | {<key>:{$ne:<value>}} | db.col.find({"likes":{$ne:50}}).pretty() | where likes != 50 |

## 基本语句
1. 数据库操作
  * 创建数据库：use DATABASE_NAME
  * 查看数据库：show dbs
  * 删除数据库：db.dropDatabase()

2. 集合操作
  * 创建集合：db.createCollection(COLLECTION_NAME)
  * 查看集合：show collections
  * 删除集合：db.COLLECTION_NAME.drop()

3. 文档操作
  * 插入文档：db.COLLECTION_NAME.insert(document)
  * 查看文档：db.COLLECTION_NAME.find(query, projection)
  * 更新文档：db.COLLECTION_NAME.update(document) /db.COLLECTION_NAME.save(document)
  * 删除文档：db.COLLECTION_NAME.remove(document)
4. 其他
  * limit()、skip()、sort()、创建索引：ensureIndex()
  * 聚合 db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)

## 应用场景
* 日志分析
* 存储用户不敏感信息、评论信息
* 工单系统
...

## 备注
CAP:
* 一致性(Consistency) (所有节点在同一时间具有相同的数据)
* 可用性(Availability) (保证每个请求不管成功或者失败都有响应)
* 分隔容忍(Partition tolerance) (系统中任意信息的丢失或失败不会影响系统的继续运作)

## 参考
1. [http://www.runoob.com/mongodb/mongodb-tutorial.html](http://www.runoob.com/mongodb/mongodb-tutorial.html)
2. [http://blog.csdn.net/xiaoxiong_web/article/details/53404428](http://blog.csdn.net/xiaoxiong_web/article/details/53404428)