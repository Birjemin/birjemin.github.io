# FormData对象上传

## encypt方式
* Method: POST; Encoding type: application/x-www-form-urlencoded (default):

>Content-Type: application/x-www-form-urlencoded
>
>foo=bar&baz=The+first+line.%0D%0AThe+second+line.%0D%0A

* Method: POST; Encoding type: text/plain:

>Content-Type: text/plain
>
>foo=bar
>baz=The first line.
>The second line.

* Method: POST; Encoding type: multipart/form-data:

>Content-Type: multipart/form-data; boundary=---------------------------314911788813839
>
>-----------------------------314911788813839
>Content-Disposition: form-data; name="foo"
>
>bar
>-----------------------------314911788813839
>Content-Disposition: form-data; name="baz"
>
>The first line.
>The second line.
>
>-----------------------------314911788813839--

* However, if you are using the GET method, a string like the following will be simply added to the URL:

>?foo=bar&baz=The%20first%20line.%0AThe%20second%20line.

request payload
## 参考
1. [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Submitting_forms_and_uploading_files](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Submitting_forms_and_uploading_files)