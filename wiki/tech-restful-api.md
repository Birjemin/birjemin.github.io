# Restful Api

## 简介
Resource Representational State Transfer(不知是啥)

![Restful Api](http://upload.ouliu.net/i/2018020201183875q9j.jpeg)

## 特征
(1) 每一个uri代表一种资源；

(2) 客户端和服务器之间，传递这种资源的某种表现层；

(3) 客户端通过四个HTTP动词(GET、POST、DELETE、PUT)，对服务器端资源进行操作，实现"表现层状态转化"(增删改查)。

(4) URL中通常不出现动词，只有名词

(5) 使用JSON不使用XML

## 设计方式

|动词 | 描述  |
|:-------:|:-------------:|
|HEAD（SELECT）| 只获取某个资源的头部信息 |
|GET（SELECT）|	获取资源 |
|POST（CREATE）|	创建资源 |
|PATCH（UPDATE）|	更新资源的部分属性（很少用，一般用POST代替）|
|PUT（UPDATE）|	更新资源，客户端需要提供新建资源的所有属性 |
|DELETE（DELETE）| 删除资源 |

使用方式
```
GET http://www.birjemin.com/api/user # 获取列表
POST http://www.birjemin.com/api/user # 创建用户
PUT http://www.birjemin.com/api/user/{id} # 修改用户信息
DELETE http://www.birjemin.com/api/user/{id} # 删除用户信息
```

## 过滤信息
用于补充规范一些通用字段

```
?limit=10 # 指定返回记录的数量
?offset=10 # 指定返回记录的开始位置
?page=2&per_page=100 # 指定第几页，以及每页的记录数
?sortby=name&order=asc # 指定返回结果按照哪个属性排序，以及排序顺序
?state=close # 指定筛选条件
```

## 状态码
字段名称为：code

|状态码 | 状态  | 方式  | 描述  |
|:-------:|:-------------:|:-------------:|:-------------:|
|200 | OK | [GET] | 服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）|
|201 | CREATED | [POST/PUT/PATCH]：用户新建或修改数据成功 |
|202 | ACCEPTED | [*]：表示一个请求已经进入后台排队（异步任务） |
|204 | NO CONTENT | [DELETE]：用户删除数据成功 |
|400 | INVALID REQUEST | [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的 |
|401 | UNAUTHORIZED | [*]：表示用户没有权限（令牌、用户名、密码错误） |
|403 | FORBIDDEN | [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的 |
|404 | NOT FOUND | [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的 |
|406 | NOT ACCEPTABLE | [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式） |
|410 | GONE | [GET]：用户请求的资源被永久删除，且不会再得到的 |
|422 | UNPROCESABLE ENTITY | [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误 |
|500 | INTERNAL SERVER ERROR | [*]：服务器发生错误，用户将无法判断发出的请求是否成功 |

## 错误信息
字段名称：message

## 示例
```
{
"code": 200,
"message": "啊哈哈",
"succ": true,
"data": []
}
```

## 备注
为了简便，有时候code没有，或者为0，message换成msg,这些实际上实在实际项目中的，不过还是要有一个规范出来，这样方便统一管理。

## 参考
1. [https://www.cnblogs.com/imyalost/p/7923230.html](https://www.cnblogs.com/imyalost/p/7923230.html)
2.[http://www.ruanyifeng.com/blog/2014/05/restful_api](http://www.ruanyifeng.com/blog/2014/05/restful_api)
