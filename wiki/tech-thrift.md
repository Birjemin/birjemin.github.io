# Thrift

## 简介

```
The Apache Thrift software framework, for scalable cross-language services development, combines a software stack with a code generation engine to build services that work efficiently and seamlessly between C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, OCaml and Delphi and other languages.
```
Apache Thrift是一个软件框架，用来进行可扩展跨语言的服务开发，结合了软件堆栈和代码生成引擎，用来构建C++,Java,Python...等语言，使之它们之间无缝结合、高效服务。

## 安装

```
brew install thrift
```

## 作用
> 跨语言调用，打破不同语言之间的隔阂。
> 跨项目调用，微服务的么么哒。

## 示例
### 前提
1.thrift版本：

![http://upload.ouliu.net/i/20180223002626ad08g.png](http://upload.ouliu.net/i/20180223002626ad08g.png)

2.Go的版本、Php版本、Python版本：

![http://upload.ouliu.net/i/20180223012029xj2s8.jpeg](http://upload.ouliu.net/i/20180223012029xj2s8.jpeg)

### 说明
该示例包含python,php,go三种语言。（java暂无）

### 新建HelloThrift.thrift
1.进入目录

```
cd /Users/birjemin/Developer/Php/study-php
```

2.编写HelloThrift.thrift

```
vim HelloThrift.thrift
```
内容如下：

```
namespace php HelloThrift                                    
  string SayHello(1:string username)
}
```

### Php测试
1.加载thrift-php库
```
composer require apache/thrift
```

2.生成php版本的thrift相关文件

```
cd /Users/birjemin/Developer/Php/study-php
thrift -r --gen php:server HelloThrift.thrift
```
这时目录中生成了一个叫做`gen-php`的目录。

3.建立`Server.php`文件

```
<?php
/**
 * Created by PhpStorm.
 * User: birjemin
 * Date: 22/02/2018
 * Time: 3:59 PM
 */

namespace HelloThrift\php;
require_once 'vendor/autoload.php';

use Thrift\ClassLoader\ThriftClassLoader;
use Thrift\Protocol\TBinaryProtocol;
use Thrift\Transport\TPhpStream;
use Thrift\Transport\TBufferedTransport;

$GEN_DIR = realpath(dirname(__FILE__)).'/gen-php';
$loader  = new ThriftClassLoader();
$loader->registerDefinition('HelloThrift',$GEN_DIR);
$loader->register();

class HelloHandler implements \HelloThrift\HelloServiceIf
{
    public function SayHello($username)
    {
        return "Php-Server: " . $username;
    }
}

$handler   = new HelloHandler();
$processor = new \HelloThrift\HelloServiceProcessor($handler);
$transport = new TBufferedTransport(new TPhpStream(TPhpStream::MODE_R | TPhpStream::MODE_W));
$protocol  = new TBinaryProtocol($transport,true,true);
$transport->open();
$processor->process($protocol,$protocol);
$transport->close();
```

4.建立`Client.php`文件

```
<?php
/**
 * Created by PhpStorm.
 * User: birjemin
 * Date: 22/02/2018
 * Time: 4:00 PM
 */

namespace  HelloThrift\php;
require_once 'vendor/autoload.php';

use Thrift\ClassLoader\ThriftClassLoader;
use Thrift\Protocol\TBinaryProtocol;
use Thrift\Transport\TSocket;
use Thrift\Transport\THttpClient;
use Thrift\Transport\TBufferedTransport;
use Thrift\Exception\TException;

$GEN_DIR = realpath(dirname(__FILE__)).'/gen-php';
$loader  = new ThriftClassLoader();
$loader->registerDefinition('HelloThrift', $GEN_DIR);
$loader->register();

if (array_search('--http',$argv)) {
    $socket = new THttpClient('local.study-php.com', 80,'/Server.php');
} else {
    $host = explode(":", $argv[1]);
    $socket = new TSocket($host[0], $host[1]);
}
$transport = new TBufferedTransport($socket,1024,1024);
$protocol  = new TBinaryProtocol($transport);
$client    = new \HelloThrift\HelloServiceClient($protocol);
$transport->open();
echo $client->SayHello("Php-Client");
$transport->close();
```

5.测试

```
php Client.php --http
```

![http://upload.ouliu.net/i/201802230042026zg6h.png](http://upload.ouliu.net/i/201802230042026zg6h.png)

### Python测试
1.加载thrift-python3模块（只测试python3,python2就不测试了）

```
pip3 install thrift
```

2.生成python3版本的thrift相关文件

```
thrift -r --gen py HelloThrift.thrift
```
这时目录中生成了一个叫做`gen-py`的目录。

3.建立`Server.py`文件

```
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import sys
sys.path.append('./gen-py')

from HelloThrift import HelloService
from HelloThrift.ttypes import *
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol
from thrift.server import TServer

class HelloWorldHandler:
    def __init__(self):
        self.log = {}

    def SayHello(self, user_name = ""):
        return "Python-Server: " + user_name

handler = HelloWorldHandler()
processor = HelloService.Processor(handler)
transport = TSocket.TServerSocket('localhost', 9091)
tfactory = TTransport.TBufferedTransportFactory()
pfactory = TBinaryProtocol.TBinaryProtocolFactory()
server = TServer.TSimpleServer(processor, transport, tfactory, pfactory)
server.serve()
```

4.建立`Client.py`文件

```
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import sys
sys.path.append('./gen-py')

from HelloThrift import HelloService
from HelloThrift.ttypes import *
from HelloThrift.constants import *

from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol

host = sys.argv[1].split(':')
transport = TSocket.TSocket(host[0], host[1])
transport = TTransport.TBufferedTransport(transport)
protocol = TBinaryProtocol.TBinaryProtocol(transport)
client = HelloService.Client(protocol)
transport.open()

msg = client.SayHello('Python-Client')
print(msg)
transport.close()
```

5.测试

开一个新窗口，运行下面命令：

```
python3 Server.php
```

![http://upload.ouliu.net/i/20180223010050l7322.png](http://upload.ouliu.net/i/20180223010050l7322.png)

### Go测试
1.加载go的thrift模块

```
go get git.apache.org/thrift.git/lib/go/thrift
```

2.生成go版本的thrift相关文件

```
thrift -r --gen go HelloThrift.thrift
```

3.生成`Server.go`文件

```
package main

import (
	"./gen-go/hellothrift"
	"git.apache.org/thrift.git/lib/go/thrift"
	"context"
)

const (
	NET_WORK_ADDR = "localhost:9092"
)

type HelloServiceTmpl struct {
}

func (this *HelloServiceTmpl) SayHello(ctx context.Context, str string) (s string, err error) {
	return "Go-Server:" + str, nil
}

func main() {
	transportFactory := thrift.NewTTransportFactory()
	protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()
	transportFactory = thrift.NewTBufferedTransportFactory(8192)
	transport, _ := thrift.NewTServerSocket(NET_WORK_ADDR)
	handler := &HelloServiceTmpl{}
	processor := hellothrift.NewHelloServiceProcessor(handler)
	server := thrift.NewTSimpleServer4(processor, transport, transportFactory, protocolFactory)
	server.Serve()
}
```

4.生成`Client.go`文件

```
package main

import (
	"./gen-go/hellothrift"
	"git.apache.org/thrift.git/lib/go/thrift"
	"fmt"
	"context"
	"os"
)

func main() {
	var transport thrift.TTransport
	args := os.Args
	protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()
	transport, _ = thrift.NewTSocket(args[1])
	transportFactory := thrift.NewTBufferedTransportFactory(8192)
	transport, _ = transportFactory.GetTransport(transport)
	//defer transport.Close()
	transport.Open()
	client := hellothrift.NewHelloServiceClientFactory(transport, protocolFactory)
	str, _ := client.SayHello(context.Background(), "Go-Client")
	fmt.Println(str)
}
```

5.测试
 
开一个新窗口，运行下面命令：

```
go run Server.go
```

![http://upload.ouliu.net/i/20180223010517jgc30.png](http://upload.ouliu.net/i/20180223010517jgc30.png)

### 综合测试
1.开启两个窗口，保证server开启

```
go run Server.go # localhost:9092
```

```
python3 Server.py # localhost:9091
```

2.php调用go-server、py-server

```
php Client.php localhost:9091
```

```
php Client.php localhost:9092
```

3.python3调用go-server、py-server

```
python3 Client.py localhost:9091
```

```
python3 Client.py localhost:9092
```

4.go调用go-server、py-server

```
go run Client.go localhost:9091
```

```
go run Client.go localhost:9092
```

## 备注
1.没有测试php的socket，go\python3的http，可以花时间做一下
2.代码纯属组装，没有深刻了解其中原理，后期打算写一篇理论篇和封装类库篇

## 参考
1. [https://studygolang.com/articles/1120](https://studygolang.com/articles/1120)
2. [https://www.cnblogs.com/qufo/p/5607653.html](https://www.cnblogs.com/qufo/p/5607653.html)
3. [https://www.cnblogs.com/lovemdx/archive/2012/11/22/2782180.html](https://www.cnblogs.com/lovemdx/archive/2012/11/22/2782180.html)
4. [https://github.com/yuxel/thrift-examples](https://github.com/yuxel/thrift-examples)
5. [http://thrift.apache.org/](http://thrift.apache.org/)