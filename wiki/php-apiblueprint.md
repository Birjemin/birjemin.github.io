# ApiBlueprint在laravel框架中的使用

## 说明

```
API Blueprint is simple and accessible to everybody involved in the API lifecycle. Its syntax is concise yet expressive. With API Blueprint you can quickly design and prototype APIs to be created or document and test already deployed mission-critical APIs
```
通俗一点就是一种接口规范，而且是使用markdown的，使用markdown的语法书写接口。

## 背景
目前和前端联调、其他部门联调使用的方式有：

1. 使用MarkDown出接口文档，放在共同的gitlab仓库上面，前后端都可以访问（只要约定好谁修改就好了，避免两个人都修改出现差异），作为一个经常写方法注释的好程序员来说（其实你的leader也会要求你），要在每一个接口上面写上几行方法注释，注明这个方法是做啥的，不然别人接手不便捷 ^_^；

2. 使用apiDoc工具（使用方法可以查看：[http://birjemin.com/wiki/php-apidoc](http://birjemin.com/wiki/php-apidoc)），将相应的接口规范以注释的方式写在每一个方法上面，然后生成相应的apiDoc文档（只需要写一下注释，不需要再写接口文档啦~），通过node服务搭建一个服务环境，前端直接访问我的开发机进行查看啦~

3. 使用`birjemin/blueprint`+`aglio`/`macdown`...组合，比如在接口的方法上面按照一定的格式进行注释，然后使用该composer包生成apib文件，这个文件是一个遵循blueprint接口规范的markdown文件。可以使用aglio插件搭建一个web服务（这个插件支持实时更新，不需要刷新页面），也可以使用macdown编辑器查看这个文件。

## 搭建步骤

![npm版本](http://upload.ouliu.net/i/20180323150537lblhz.png)

1.安装aglio（npm是啥？？？自己问前端同学吧。。）

```
npm install -g aglio
```
请检查是否安装成功。

2.给laravel项目引入composer包（包已经提交，不过国内镜像还没同步）

```
composer require birjemin/blueprint dev-master
```
or
```
composer require birjemin/blueprint
```

3.在`app.php`中添加`BlueprintServiceProvider::class`

4.给接口添加注释，添加在Controller入口方法前面(详细的注释方式请查看[https://github.com/dingo/api/wiki/API-Blueprint-Documentation](https://github.com/dingo/api/wiki/API-Blueprint-Documentation))。

```
  /**
  * Class CarsController
  * @package App\Http\Controllers
  * @Resource("CarResource", uri="/api/cars")
  */
  class CarController extends Controller
  {
  /**
    * cars list
    *
    * Get current cars list
    *
    * @Get("/list")
    * @Transaction({
    *      @Request(identifier="page=1&type=1"),
    *      @Response(200, body={"msg": "返回成功","code": 200,"page": 1,"timestamp": "1522673813","data":{"result":[{"price": "2200","type": "福特","notice": "豪车"},{"price": "2200","type": "大众","notice": "车"}]}})
    * })
    * @Parameters({
    *      @Parameter("page", type="integer", required=true, description="分页", default=1),
    *      @Parameter("search", type="string", required=false, description="搜索条件"),
    *      @Parameter("type", type="integer", required=true, description="汽车类型", default=1, members={
    *          @Member(value="1", description="新车"),
    *          @Member(value="2", description="旧车")
    *      })
    * })
    */
    public function index()
    {
      return json_decode('{"succ":true,"code":0,"message":"","data":{"result":true}})');
    }
  }
```

> `@Response(200, body={...}`中的`[`和`]`要替换成`{`和`}`，这里不替换是因为我的博客使用了`jekyll`,github报错啦~~

5.创建`apib`文件

```
php artisan birjemin:docs --output-file=tianming.apib
```
项目下面生成了tianming.apib文件，这个是一个markdown文件，可以直接用markdown编辑器打开，下面讲一下aglio的web服务。

6.运行aglio服务(详细的命令可以去[https://github.com/danielgtaylor/aglio](https://github.com/danielgtaylor/aglio)查看)

```
aglio -i tianming.apib -s
```

7.访问:`http://27.0.0.1:3000`(端口可以自定义，这个是默认的)

![http://upload.ouliu.net/i/2018040222280130t9l.png](http://upload.ouliu.net/i/2018040222280130t9l.png)

## 注意点
1.返回的数据不可以json如果是数组,`{}`代替`[]`这个符号，比如示例中的。

```
 {"msg": "返回成功","code": 200,"page": 1,"timestamp": "1522673813","data":{"result":[{"price": "2200","type": "福特","notice": "豪车"},{"price": "2200","type": "大众","notice": "车"}]}}
```
就要将里面的中括号替换成大括号！


2.这些注释写错了会报错哦~

3.和apidoc的注释有区别~~

## 补充文档
* [https://apiblueprint.org/](https://apiblueprint.org/)
* [https://github.com/danielgtaylor/aglio](https://github.com/danielgtaylor/aglio)
* [https://github.com/dingo/api/wiki/API-Blueprint-Documentation](https://github.com/dingo/api/wiki/API-Blueprint-Documentation)

## 参考
1. [https://apiblueprint.org/](https://apiblueprint.org/)
2. [http://www.piaoyi.org/php/DingoApi-Laravel-tips.html](http://www.piaoyi.org/php/DingoApi-Laravel-tips.html)
3. [https://github.com/danielgtaylor/aglio](https://github.com/danielgtaylor/aglio)
4. [https://github.com/dingo/blueprint](https://github.com/dingo/blueprint)
5. [https://github.com/dingo/api/wiki/API-Blueprint-Documentation](https://github.com/dingo/api/wiki/API-Blueprint-Documentation)
6. [https://github.com/monkeycraps/blueprint](https://github.com/monkeycraps/blueprint)