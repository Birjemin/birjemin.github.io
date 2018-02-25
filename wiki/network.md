#OSI七层与TCP/IP五层协议

## 简介
划分互联网结构，将互联网进行分层，规定每层的职责，制定每层所需要的协议。

## 说明
* OSI七层协议

| OSI层 | 功能 | 协议 | 
| :---: | :---: | :---: |
| 应用层 | 文件传输，电子邮件，文件服务，虚拟终端 | TFTP，HTTP，SNMP，FTP，SMTP，DNS，Telnet |
| 表示层 | 数据格式化，代码转换，数据加密 | 没有协议 |
| 会话层 | 解除或建立与别的接点的联系 | 没有协议 |
| 传输层 | 提供端对端的接口 | TCP，UDP |
| 网络层 | 为数据包选择路由 | IP，ICMP，RIP，OSPF，BGP，IGMP |
| 数据链路层 | 传输有地址的帧以及错误检测功能 | SLIP，CSLIP，PPP，ARP，RARP，MTU |
| 物理层 | 以二进制数据形式在物理媒体上传输数据 | ISO2110，IEEE802，IEEE802.2 |

* TCP/IP五层协议

> * 应用层
> * 传输层
> * 网络层
> * 数据链路层
> * 物理层

## 简介
先留个坑，以后再填。

## 参考
1. [http://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html](http://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html)
2. [http://www.ruanyifeng.com/blog/2012/06/internet_protocol_suite_part_ii.html](http://www.ruanyifeng.com/blog/2012/06/internet_protocol_suite_part_ii.html)
3.[http://blog.csdn.net/cainv89/article/details/46885197](http://blog.csdn.net/cainv89/article/details/46885197)