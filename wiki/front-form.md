# 表单属性

## 背景
整理和记录一下form的属性，方便查阅

## 简介（列出部分属性进行说明）
### MIME type(content type)

```
A media type (also MIME type and content type) is a two-part identifier for file formats and format contents transmitted on the Internet
```

MIME type是因特网上传输的标识符，它由两部分组成，一个是文件格式一个是格式化内容（format为动词）。
For example:
```
application/javascript
application/json
application/x-www-form-urlencoded
application/xml
application/zip
application/pdf
multipart/form-data
text/css
text/html
text/csv
text/plain
image/png
image/jpeg
image/gif
```

### encypt
当method为post的时候enctype的值是提交给服务器内容的MIME类型。

* Form默认的encypt[Method: POST; Encoding type: application/x-www-form-urlencoded (default)]
```
Content-Type: application/x-www-form-urlencoded

foo=bar&baz=The+first+line.%0D%0AThe+second+line.%0D%0A
```
* 纯文本形式，HTML5的属性[Method: POST; Encoding type: text/plain]

```
Content-Type: text/plain

foo=bar
baz=The first line.
The second line.
```
* Form中的`input`标签属性是`file`[Method: POST; Encoding type: multipart/form-data]

```
Content-Type: multipart/form-data; boundary=---------------------------314911788813839

-----------------------------314911788813839
Content-Disposition: form-data; name="foo"

bar
-----------------------------314911788813839
Content-Disposition: form-data; name="baz"

The first line.
The second line.

-----------------------------314911788813839--
```

* Form的method为get[Method: GET;a string like the following will be simply added to the URL]

```
?foo=bar&baz=The%20first%20line.%0AThe%20second%20line.
```

### Method
GET POST

## 注意点

1. 如果请求头里设置`Content-Type: application/x-www-form-urlencoded`，那么这个请求被认为是表单请求，参数出现在Form Data里，格式为`key=value&key=value&key=value...`
2. 原生的AJAX请求头里设置`Content-Type:application/json`，或者使用请求头`Content-Type:text/plain;`参数会显示在`Request payload`块里提交。
3. 亲测laravel中都可以通过`request()->all()`获取到内容。

## 参考
1. [https://en.wikipedia.org/wiki/Media_type](https://en.wikipedia.org/wiki/Media_type)
2. [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Submitting_forms_and_uploading_files](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Submitting_forms_and_uploading_files)
3. [https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)
4. [https://github.com/kaola-fed/blog/issues/105](https://github.com/kaola-fed/blog/issues/105)