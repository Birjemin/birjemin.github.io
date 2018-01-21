# PHP变量的实现

## 简介
在官方的PHP实现内部，所有变量使用同一种数据结构(zval)来保存，而这个结构同时表示PHP中的各种数据类型。 它不仅仅包含变量的值，也包含变量的类型。这就是PHP弱类型的核心。
* 标量类型： boolean、integer、float(double)、string
* 复合类型： array、object
* 特殊类型： resource、NULL

## 变量的存储结构
PHP在内核中是通过zval这个结构体来存储变量的，它的定义在Zend/zend.h文件里，简短精炼，只有四个成员组成：
```
struct _zval_struct {
    zvalue_value value; /* 变量的值 */
    zend_uint refcount__gc;
    zend_uchar type;    /* 变量当前的数据类型 */
    zend_uchar is_ref__gc;
};
typedef struct _zval_struct zval;
//在Zend/zend_types.h里定义的：
typedef unsigned int zend_uint;
typedef unsigned char zend_uchar;
```
* refcount__gc	表示引用计数	1
* is_ref__gc	表示是否为引用	0
* value	存储变量的值	
* type	变量具体的类型	

## 变量的类型
zval结构体的type字段就是实现弱类型最关键的字段，type的值可以为： IS_NULL、IS_BOOL、IS_LONG、IS_DOUBLE、IS_STRING、IS_ARRAY、IS_OBJECT、IS_RESOURCE。 从字面上就很好理解，他们只是类型的唯一标示，根据类型的不同将不同的值存储到value字段。 除此之外，和他们定义在一起的类型还有IS_CONSTANT和IS_CONSTANT_ARRAY。

## 示例
```
<?php
$foo = 'bar';
?>
```    
上面是一段PHP语言的例子，创建一个变量，并把它的值设置为'bar'，步骤：
创建一个zval结构，并设置其类型。
设置值为'bar'。
将其加入当前作用域的符号表，这样用户才能在PHP里使用这个变量
具体的代码为：
```
{
    zval *fooval;
    MAKE_STD_ZVAL(fooval);
    ZVAL_STRING(fooval, "bar", 1);
    ZEND_SET_SYMBOL( EG(active_symbol_table) ,  "foo" , fooval);
}    
```   
首先，我们声明一个zval指针，并申请一块内存。然后通过ZVAL_STRING宏将值设置为`bar`,最后一行的作用就是将这个zval加入到当前的符号表里去，并将其label定义成foo，这样用户就可以在代码里通过$foo来使用它。

## 备注
建议好好看看参考2的链接，里面讲了很多关于变量的知识。

## 参考
1. [http://www.laruence.com/2008/08/12/180.html](http://www.laruence.com/2008/08/12/180.html)
2. [https://www.kancloud.cn/nickbai/php7/363267](https://www.kancloud.cn/nickbai/php7/363267)