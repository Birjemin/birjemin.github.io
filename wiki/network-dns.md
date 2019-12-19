# DNS

## 释义
```
The Domain Name System (DNS) is a distributed directory that resolves human-readable hostnames, such as www.dyn.com, into machine-readable IP addresses like 50.16.85.103. DNS is also a directory of crucial information about domain names, such as email servers (MX records) and sending verification (DKIM, SPF, DMARC), TXT record verification of domain ownership, and even SSH fingerprints (SSHFP).
```
域名系统（DNS）是一个分布式的目录，主要要是将人类可读的域名转成对应的IP地址。

## 简介

### 类比
DNS相当于一个电话簿，当你想给小A（相当于域名）打电话（访问网站），你只需要在电话簿中找到他的名称，直接拨号就好了，而不需要去记住他的电话是多少（ip地址）。

### 名词解释

1. DNS Resolver：DNS递归解析器，DNS解析域名IP地址的第一步，可以说当我们访问某一个域名时，系统会到这里来获取域名对应的IP地址，然后进行访问，一般默认是网络提供商（即：internet service provider (ISP)）提供的。也可以在网卡设置中改用其他提供商，比如Google Public DNS，OpenDNS...

2. Root Nameserver：根域名服务器，管理互联网的主目录，相当于一个索引服务器，IPv4全球有13台。如果`DNS递归解析器`在缓存中没有找到域名的IP地址，或者缓存已经失效，则`DNS递归解析器`会去`根域名服务器`查询当前域名需要去哪个`顶级域名服务器`可以得到存储该域名的IP地址的`权威域名服务器`。

3. Top Level Domain Server：顶级域名服务器，管理互联网的不同域名的目录（比如com，org，cn...），`DNS递归解析器`会在这里找到存储域名IP地址的`权威域名服务器`。

4. Authoritative Nameserver：权威域名服务器，存储域名IP地址的服务器

### 工作方式

1. A user types ‘example.com’ into a web browser and the query travels into the Internet and is received by a DNS recursive resolver.

2. The resolver then queries a DNS root nameserver (.).

3. The root server then responds to the resolver with the address of a Top Level Domain (TLD) DNS server (such as .com or .net), which stores the information for its domains. When searching for example.com, our request is pointed toward the .com TLD.

4. The resolver then makes a request to the .com TLD.

5. The TLD server then responds with the IP address of the domain’s nameserver, example.com.

6. Lastly, the recursive resolver sends a query to the domain’s nameserver.

7. The IP address for example.com is then returned to the resolver from the nameserver.

8. The DNS resolver then responds to the web browser with the IP address of the domain requested initially.Once the 8 steps of the DNS lookup have returned the IP address for example.com, the browser is able to make the request for the web page:

9. The browser makes a HTTP request to the IP address.

10. The server at that IP returns the webpage to be rendered in the browser (step 10).

![2019121901.png](./../assets/images/2019121901.png)

## 备注
其实挺好理解的，简单的总结吧。（参考4是中文版本的翻译）

## 参考
1. [https://www.cloudflare.com/learning/dns/what-is-dns/](https://www.cloudflare.com/learning/dns/what-is-dns/)
2. [https://dyn.com/blog/dns-why-its-important-how-it-works/](https://dyn.com/blog/dns-why-its-important-how-it-works/)
3. [http://www.ruanyifeng.com/blog/2016/06/dns.html](http://www.ruanyifeng.com/blog/2016/06/dns.html)
4. [https://mp.weixin.qq.com/s/yNUSo_lx_yR0tffqOgXMkQ](https://mp.weixin.qq.com/s/yNUSo_lx_yR0tffqOgXMkQ)