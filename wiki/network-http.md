# HTTP协议

## 简介
```
The Hypertext Transfer Protocol (HTTP) is an application protocol for distributed, collaborative, hypermedia information systems. HTTP is the foundation of data communication for the World Wide Web.
```
超文本传输协议是用于分布式，协作式和超媒体信息系统的应用层协议。HTTP协议是万维网数据通信的基础。
设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。通过HTTP或者HTTPS协议请求的资源由统一资源标识符（Uniform Resource Identifiers，URI）来标识。
通常，由HTTP客户端发起一个请求，创建一个到服务器指定端口（默认是80端口）的TCP连接。HTTP服务器则在那个端口监听客户端的请求。一旦收到请求，服务器会向客户端返回一个状态，比如"HTTP/1.1 200 OK"，以及返回的内容，如请求的文件、错误消息、或者其它信息。

```
          userinfo          host        port
        ┌───────┴───────┐ ┌────┴────────┐ ┌┴┐
 http://john.doe:password@www.example.com:123/forum/questions/?tag=networking&order=newest#top
 └─┬─┘ └───────────┬────────────────────────┘└─┬─────────────┘└────────┬──────────────────┘└┬─┘
 scheme         authority                      path                  query             fragment
```
### 版本历程
- HTTP/0.9
  已过时。只接受GET一种请求方法，没有在通讯中指定版本号，且不支持请求头。由于该版本不支持POST方法，因此客户端无法向服务器传递太多信息。
- HTTP/1.0
  这是第一个在通讯中指定版本号的HTTP协议版本，至今仍被广泛采用，特别是在代理服务器中。
- HTTP/1.1
  持久连接被默认采用，并能很好地配合代理服务器工作。还支持以管道方式在同时发送多个请求，以便降低线路负载，提高传输速度。
  HTTP/1.1相较于HTTP/1.0协议的区别主要体现在：
  ```
    缓存处理
    带宽优化及网络连接的使用
    错误通知的管理
    消息在网络中的发送
    互联网地址的维护
    安全性及完整性
  ```
- HTTP/2
  2015年提出，截止2020年1月，前1千万网站中，已经有42.6%支持HTTP2。引入了协商机制（允许Client和Server选择HTTP1.1、HTTP2.0、其他方式传输数据），减少延迟提升加载速度（比如HTTP的头部压缩，服务端推送，请求流水线化，多路复用）等。（不多解释，免得扯远了，见参考3，4）

- HTTP/3
  2018年提出，使用QUIC替代HTTP(不多解释，免得扯远了)

### 请求方法
- GET
  请求资源
- HEAD
  和GET一样，不过不返回Response Body
- POST
  提交数据
- PUT
  向指定的资源进行修改，如果资源不存在，则创建
- DELETE
  删除指定的资源
- TRACE
  回显服务器收到的请求，主要用于测试或诊断
- OPTIONS
  返回指定资源支持的请求方法，多用于测试指定资源的服务器是否正常工作
- CONNECT
  HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。通常用于SSL加密服务器的链接（经由非加密的HTTP代理服务器）。(看不明白)
- PATCH
  对指定的资源进行局部修改

### 请求流程
1. Client(或者浏览器)发起一个HTTP请求
2. Server服务端收到一个HTTP请求
3. Server调用应用服务来处理这个请求
4. Server返回一个HTTP响应给Client
5. Client接受到响应

### 请求示例
- Client请求

```
GET / HTTP/1.1
Host: www.example.com
```

- Server响应

```
HTTP/1.1 200 OK
Date: Mon, 23 May 2005 22:38:34 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 138
Last-Modified: Wed, 08 Jan 2003 23:11:55 GMT
Server: Apache/1.3.3.7 (Unix) (Red-Hat/Linux)
ETag: "3f80f-1b6-3e1cb03b"
Accept-Ranges: bytes
Connection: close

<html>
  <head>
    <title>An Example Page</title>
  </head>
  <body>
    <p>Hello World, this is a very simple HTML document.</p>
  </body>
</html>
```
### 状态码
- 1xx消息——请求已被服务器接收，继续处理
- 2xx成功——请求已成功被服务器接收、理解、并接受
- 3xx重定向——需要后续操作才能完成这一请求
- 4xx请求错误——请求含有词法错误或者无法被执行
- 5xx服务器错误——服务器在处理某个正确请求时发生错误

## 参考
1. [https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
2. [https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)
3. [https://en.wikipedia.org/wiki/HTTP/2](https://en.wikipedia.org/wiki/HTTP/2)
4. [https://juejin.im/post/5a4dfb2ef265da43305ee2d0](https://juejin.im/post/5a4dfb2ef265da43305ee2d0)