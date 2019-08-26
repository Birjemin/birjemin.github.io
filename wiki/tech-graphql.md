# 30分钟从RESTful到GraphQL

## Q:前端和后端的通讯方式？
webSocket、表单提交、jsonp、ajax...

## API
API（Application Programming Interface，应用程序编程接口）是一些
预先定义的函数，目的是提供应用程序与开发人员基于某软件或硬件得以访问一组例程的能力，屏蔽底层细节。

## RESTful
1. 定义：Representational State Transfer(表现层【资源】状态转化)。
2. 特点：
    * 每一个URI代表一种资源；
    * 客户端和服务器之间，传递这种资源的某种表现层；
    * 客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"；
3. 规范：
```
【GET】     /{resources}/{resource_id}      // 返回单个资源对象
【GET】     /{resources}                    // 返回资源对象的列表
【POST】    /{resources}                    // 返回新生成的资源对象
【PUT】     /{resources}/{resource_id}      // 返回完整的资源对象
【PATCH】   /{resources}/{resource_id}      // 返回完整的资源对象
【DELETE】  /{resources}/{resource_id}      // 状态码 200，返回完整的资源对象。
```
4. 示例：

* 对用户和订单的增删改查

```
# 前台
GET getUserInfo.php
POST addUserInfo.php
POST updateUserInfo.php
POST|GET deleteUserInfo.php

GET getOrderInfo.php
POST addOrderInfo.php
POST updateOrderInfo.php
POST|GET deleteOrderInfo.php

# 后台
GET getUserOrderInfo.php
...
```

```
# 前台
GET api/user
PUT api/user
DELETE api/user
POST api/user

GET api/orders
GET api/orders/{order_id}
PUT api/orders/{order_id}
DELETE api/orders/{order_id}
POST api/orders

# 后台
GET api/users/{user_id}/orders
GET api/users/{user_id}/orders/{order_id}
PUT api/users/{user_id}/orders/{order_id}
DELETE api/users/{user_id}/orders/{order_id}
POST api/users/{user_id}/orders
```

Q: 如果接口存在版本控制，该如何处理？？

* 获取资源

```
GET api/user
GET api/banner-lists
GET api/hobbies
```

5. Laravel对Restful的实现，参考13

6. 优缺点
    * 并不是所有的东西都是资源（多选批量更改，批量删除，n个delete请求？？？支付回调方法）。
    * 仅需要少量资源，返回了整体资源(数据定制问题，客户端对数据没有控制权)。
    * 未定义返回数据的格式（string? json? null? object? 异常处理问题）。
    * 有时一个页面调用api太多，开销高（多次请求问题，涉及到资源的整合）。
    * 产品迭代给api带来的挑战。
    * ...

## GraphQL
1. 背景：Facebook开发GraphQL的最初原因是移动用户的增加、低功耗设备和松散的网络。GraphQL最小化了需要网络传输的数据量，从而极大地改善了在这些条件下运行的应用程序。
2. 定义：GraphQL 既是一种用于 API 的查询语言也是一个满足你数据查询的运行时。GraphQL 对你的 API 中的数据提供了一套易于理解的完整描述，使得客户端能够准确地获得它需要的数据，而且没有任何冗余，也让 API 更容易地随着时间推移而演进，还能用于构建强大的开发者工具(参考12)。
```
GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data.
```

<b>GraphQL试图抽象服务端的数据成为一个综合的数据集合，而前端（客户端）的请求就是查询数据库。</b>

3. 示例：

![官方示例](http://thyrsi.com/t6/376/1537872909x-1566679839.png)

4. 优缺点：
    * 取消服务端硬编码（面向客户端友好【结对】编程，减少对服务端的依赖）。
    * http传输数据减少（request，response）。
    * 强类型。
    * 服务端性能（复杂的查询会导致资源消耗）。
    * 客户端缓存（restful通过url进行缓存数据，而GraphQL失去了这个能力）、服务器缓存面临挑战。
    * N+1 问题（多对一，多对多模型中需要循环sql查询，直至找到需求的数。[6.1](http://jerryzou.com/posts/10-questions-about-graphql/)）。

## RESTful vs GraphQL

|  | 耦合性 | 约束性 | 复杂度 | 缓存 | 可发现性 | 版本控制 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| REST(Resource) | low | low | low | http | good | easy |
| GraphQL(Query) | medium | high | medium | custom | good | ??? |

## 延伸
1. GraphQL(参考9)
```
// require composer
composer require rebing/graphql-laravel
// 添加Provider、Facades到app/config/app.php
Rebing\GraphQL\GraphQLServiceProvider::class
'GraphQL' => Rebing\GraphQL\Support\Facades\GraphQL::class,
//生成配置文件
php artisan vendor:publish --provider="Rebing\GraphQL\GraphQLServiceProvider"
// 生成User、Job的Model，多对多关系
// 创建 GraphQL 的 Query 和 Type
// 配置文件配置query和type
// 请求。。。参考10
```
2. 面向资源编程。(参考7,8)
3. 所见即所得(What You See Is What You Get)。

## 总结
1. 认识RESTful和GraphQL。
2. 抽象能力：资源（面向资源编程，使用动作类），图（数据的表现形式最直观的表现方法）

## 参考

1. [http://jerryzou.com/posts/10-questions-about-graphql/](http://jerryzou.com/posts/10-questions-about-graphql/)
2. [https://zhuanlan.zhihu.com/p/26522359](https://zhuanlan.zhihu.com/p/26522359)
3. [https://segmentfault.com/a/1190000013961872](https://segmentfault.com/a/1190000013961872)
4. [http://www.infoq.com/cn/news/2017/07/graphql-vs-rest](http://www.infoq.com/cn/news/2017/07/graphql-vs-rest)
5. [http://www.infoq.com/cn/articles/rest-apis-are-rest-in-peace-apis-long-live-graphql](http://www.infoq.com/cn/articles/rest-apis-are-rest-in-peace-apis-long-live-graphql)
6. [http://slides.com/howtomakeaturn/model#/](http://slides.com/howtomakeaturn/model#/)
7. [https://oomusou.io/laravel/architecture/](https://oomusou.io/laravel/architecture/)
8. [https://laravel-china.org/topics/12791/laravel-program-architecture-design-idea-use-action-class](https://laravel-china.org/topics/12791/laravel-program-architecture-design-idea-use-action-class)
9. [https://laravel-china.org/articles/8115/using-graphql-one-in-laravel-get-data](https://laravel-china.org/articles/8115/using-graphql-one-in-laravel-get-data)
10. [http://graphql.cn/graphql-js/graphql-clients/](http://graphql.cn/graphql-js/graphql-clients/)
11. [https://github.com/TommyLemon/APIJSON/wiki](https://github.com/TommyLemon/APIJSON/wiki)
12. [https://insights.thoughtworks.cn/tag/graphql/](https://insights.thoughtworks.cn/tag/graphql/)
13. [https://laravel-china.org/courses/laravel-specification/502/router](https://laravel-china.org/courses/laravel-specification/502/router)