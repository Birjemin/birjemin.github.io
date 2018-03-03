# PHP开发小结
总参考并且总结。

## 功能设计
1. 功能尽量模块化；
2. 类的设计尽量单一，函数功能单一；
3. 需要uml图（类图、时序图、组件图）、文档沉淀；

## 编写风格
1. 遵循PSR-1 基础编码规范[https://laravel-china.org](https://laravel-china.org/topics/2078/psr-specification-psr-1-basic-coding-specification)
2. 遵循PSR-2 编码风格规范[https://laravel-china.org](https://laravel-china.org/topics/2079/psr-specification-psr-2-coding-style-specification)
3. 注释！！！
4. 常量、env代替硬编码
5. 路由遵循RestApi原则 
6. created_at(INT类型),updated_at(TIMESTAMP on CURRENT_TIMESTAMP，日期类型，自动更新), deleted_at(default NULL)

## 代码分布
1. MVC
* Model - sql处理，不应当出现业务
* View - 
* Controller - 处理参数验证和方法调用(更彻底的方式是新建一个请求类，请求类处理验证和参数过滤，控制类调用请求类)
* Service - 业务

2. 升级版本
[http://birjemin.com/wiki/php-ouran](http://birjemin.com/wiki/php-ouran)

## Laravel一些注意点
主要是参考2，详情请查看参考2的链接。
1. 优先使用 Eloquent 和集合来操作和处理数据。
2. 使用被社区接受的标准 Laravel 工具
3. 遵循 Laravel 命名约定
4. 使用缩写或可读性更好的语法(optional、)
5. 使用 IoC 容器或门面
6. 不要从直接从 .env 获取数据(采用config()方法)

## 需求流程
1. 需求评审会议
2. QA出Case，DEV估时、出方案（类图、时序图、组件图）
3. 评审需求Case会议、需求设计方案会议
4. 开发
5. DEV根据Case自测（应该还有一个单元测试）
6. 交付QA（功能测试）
7. 验收
8. 上线
9. 复盘会议

## 和产品PD撕逼的艺术

能动手就别动嘴

## 补充
要把产品经理PM拉倒自己的阵营~~~

## 参考
1. [https://psr.phphub.org/](https://psr.phphub.org/)
2. [https://github.com/nonfu/laravel-best-practices](https://github.com/nonfu/laravel-best-practices)
3. [http://plantuml.com/](http://plantuml.com/)