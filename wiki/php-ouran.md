# ouran

## 简介
只能支撑一些小的接口，比如小程序接口，webapp接口等

## 结构
```
app |
    |- Comps |
             |--- Article |
                          |--- ArticleMapper.php
                          |--- ArticleModule.php
                          |--- ArticleRepository.php
                          |--- ArticleService.php
                          |--- ArticleTrait.php
    |- doc |
           |--- sql |
                    |--- sql
    |- Config |
              |--- database.php
    |- Helpers |
               |--- Support.php
    |- Models |
              |--- Article.php
https |
      |- Controllers |
                     |--- Article |
                                  |--- ArticleController.php
                     |--- register.php
                     |--- routes.php
vendor |...
              
```

## 目录说明

* app为项目目录：
    * Comps为模块目录，将项目分成相应的模块；
        * Mapper方法来格式化输出数据（对外）；
        * Module文件为模块入口（对外）；
        * Repository为数据库操作Model的仓库（对内）；
        * Service为处理复杂逻辑（对内）；
        * Trait为对外暴露Module和Mapper文件出口；
    * Config为配置目录
    * Helpers为支持文件目录
    * Models为数据库Model目录

* doc为文档目录

* https为访问目录：
    * Controllers为路由目录；
    * routes.php为路由文件；

* vendor

## 规则

```
1.Controller只能通过Trait访问模块，Trait只能暴露Module和Mapper这两个对外的文件；
2.Model只能被Repository调用；
3.Repository和Service只能被对应模块中的Module调用，不能对外；
4.跨模块调用只能调用模块的Module，不能调用模块里面其他对象
```
## Location

项目位置：[https://github.com/Birjemin/ouran](https://github.com/Birjemin/ouran)

参考：[Flight](http://flightphp.com/)


## PS
flight还没有看完~~~~继续看吧