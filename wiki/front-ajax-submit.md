# Ajax提交数据
## 背景

1. 在业务上面使用到了ajax上传图片，采用的方式是提供一个公用的api进行图片上传，然后返回图片的在服务器的url，这样在其他地方使用到时，直接提交图片的url，而不是图片资源，避免影响应能和体验，也方便后期切换（如果后期采用了第三方图片服务，或者对图片需要进行处理，只要改造这个接口就好了）。
2. 使用Ajax提交表单数据（包含上传文件）有两种形式
  * using only AJAX
  * using the FormData API
3. 优缺点
  * Using the FormData API is the simplest and fastest, but has the disadvantage that data collected can not be stringified.
  * Using only AJAX is more complex, but typically more flexible and powerful.

## FormData简介

```
  The FormData object lets you compile a set of key/value pairs to send using XMLHttpRequest. It is primarily intended for use in sending form data, but can be used independently from forms in order to transmit keyed data. The transmitted data is in the same format that the form's submit() method would use to send the data if the form's encoding type were set to multipart/form-data.
```
  大致意思是你可以将数据使用FormData对象编译成键值对形式，然后使用XMLHttpRequest技术向后端发送数据。主要是用来发送form表单数据，也可以发送带键数据。这种形式传输的数据和一个`enctype`属性为`multipart/form-data`并且采用`submit()`方法提交的form表单传输的数据格式相同。

## Ajax使用FormData提交数据
  只是简单的示范一下文件上传，表单数据类似，不写例子了。

### Ajax上传文件-带form标签的FormData提交

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
</head>
<body>
<form id="upload" method="post">
    <input id="file" type="file" name="file"/>
    <input id="img" type="hidden"/>
    <input type="submit" value="上传图片">
</form>
<script type="text/javascript" src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
<script type="text/javascript">
var BASE_URL = '../'
$(document).ready(function(){
    $('#file').on('change', function() {
        var formData = new FormData($('#upload')[0])
        $.ajax({
            url: BASE_URL + 'api/upload',
            type: 'post',
            cache: false,
            data: formData,
            processData: false,
            contentType: false,
            success: function(res) {
                console.log(res)
            }
        })
    })
})
</script>
</body>
</html>
```

```php
<?php
print_r($_FILE);exit();
?>
```

> 特点：form表单提交,带有<form>标签，enctype="multipart/form-data"不设置也可以。

### Ajax不带form标签的FormData提交

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
</head>
<body>
    <input id="file" type="file" name="file"/>
    <input id="img" type="hidden"/>
    <input type="submit" value="上传图片">
<script type="text/javascript" src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
<script type="text/javascript">
var BASE_URL = '../'
$(document).ready(function(){
    $('#file').on('change', function() {
        console.log(this.files)
        var formData = new FormData()
        formData.append("file", this.files[0]);
        $.ajax({
            url: BASE_URL + 'api/upload',
            type: 'post',
            cache: false,
            data: formData,
            processData: false,
            contentType: false,
            success: function(res) {
                console.log(res)
            }
        })
    })
})
</script>
</body>
</html>
```

```php
<?php
print_r($_FILE);exit();
?>
```

> 没有<form>标签，基本使用场景中使用的是这种。

## Ajax不使用FormData提交数据
从参考2来看，上传文件需要使用使用FileReader对象,并且Ajax不使用FormData提交数据略复杂，幸亏有一些大咖封装了一下，比如官方提供了一个`A little vanilla framework`（一点香草？？？？？？这个使用原生写的，不是封装，，，），再比如`ajaxFileUpload`(github地址是参考5，官方有示例，试了一下，妥妥的支持IE6..)。

## 感受
FormData是HTML5新增的属性，可能在兼容浏览器上面会抛弃一些古典（old）浏览器，不过简单;利用纯Ajax提交也还好，因为有很多现成的库，比如jquery,axios...从`A little vanilla framework`的示例（参考2）来看，基本是根据form表单的encypt形式，采用相应的方式发送数据。

## 参考
1. [https://developer.mozilla.org/en-US/docs/Web/API/FormData/Using_FormData_Objects](https://developer.mozilla.org/en-US/docs/Web/API/FormData/Using_FormData_Objects)
2. [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Submitting_forms_and_uploading_files](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Submitting_forms_and_uploading_files)
3. [https://developer.mozilla.org/en-US/docs/Web/API/FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)
4. [https://www.cnblogs.com/zhuxiaojie/p/4783939.html](https://www.cnblogs.com/zhuxiaojie/p/4783939.html)
5. [https://github.com/davgothic/AjaxFileUpload](https://github.com/davgothic/AjaxFileUpload)