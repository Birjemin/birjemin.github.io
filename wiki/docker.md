# Docker入门

## 概念
```
Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers. Containers are isolated from one another and bundle their own software, libraries and configuration files; they can communicate with each other through well-defined channels. All containers are run by a single operating-system kernel and are thus more lightweight than virtual machines.
The service has both free and premium tiers. The software that hosts the containers is called Docker Engine. It was first started in 2013 and is developed by Docker, Inc.
```

Docker是一套Paas产品，该产品是一个能够提供系统级别虚拟化技术来分发软件的容器。容器之间是相互隔离的，并且只包含自己的软件、依赖和配置。容器之间可以通过定义好的渠道进行沟通，所有容器都由单个操作系统内核运行，并且比虚拟机更轻便。托管容器的软件称为Docker引擎。

## 内容
### 用途
* 持续交付你的程序
* 响应式部署和扩展
* 在同一硬件上运行更多工作负载

### Docker Engine
托管容器的软件称为Docker引擎，Docker引擎包含了三个核心组件：
  * A server which is a type of long-running program called a daemon process (the dockerd command)(守护进程服务Docker daemon)
  * A REST API which specifies interfaces that programs can use to talk to the daemon and instruct it what to do.（控制守护进程服务的接口Docker API）
  * A command line interface (CLI) client (the docker command).(CLI客户端Docker client)

![20191202301.png](../assets/images/20191202301.png)

Docker client使用的Docker API来和Docker daemon交互，交互的方式是脚本或者CLI命令。Docker程序之间交互方式是使用底层的API和CLI。Docker daemon负责创建和管理Docker对象（镜像、容器、网络、数据卷）

### Docker架构
![20191202302.svg](../assets/images/20191202302.svg)

Docker是CS架构

#### The Docker daemon（Docker daemon)
The Docker daemon (dockerd) 负责监听Docker API请求和管理Docker对象（镜像、容器、网络、数据卷）。Docker daemon还可以与其他Docker daemon通信以管理Docker服务

#### The Docker client（Docker client）
The Docker client (docker) 是用户访问Docker服务的主要方式。比如键入`docker run`命令，Docker client会发送给Docker Docker daemon来运行。命令访问Docker Docker daemon的方式是通过Docker API。Docker client可以与多个Docker daemon通信。

#### Docker registries（Docker registry）
Docker registry是储存Docker镜像的地方，比如Docker Hub就是一个公共的registry。Docker寻找镜像的默认配置是Docker Hub，当然，也可以配置自己的私有registry。当运行`docker pull`或者`docker run `命令时，docker将从你配置的registry中拉取需要的镜像来构建容器。当运行`docker push`时，你的镜像会被推送到配置的registry中。

#### Docker objects
当你使用docker的时候，你会使用到镜像、容器、网络、数据卷、扩展或者其他的对象，统称Docker objects。

##### IMAGES
Docker镜像是一个只读的模板，用来指示创建Docker容器。通常，一个镜像是在另一个镜像的基础上，添加一些定制组件来构建的。比如你想构建一个镜像，这个镜像是基于ubuntu镜像，在上面添加apache服务和你的程序，以及程序的依赖。

##### CONTAINERS
Docker容器是Docker镜像的可运行实例。可以使用Docker API或者CLI对容器进行创建、启动、停止、移除和删除。可以将一个容器连接到一个或者多个网络中、添加存储空间、甚至基于当前容器的状态创建Docker镜像。默认情况下，容器和其他容器、宿主机之间都是相互隔离的。

#### Docker Network
* bridge: 默认的网络驱动模式。如果不指定驱动程序，`bridge`便会作为默认的网络驱动模式。当应用程序运行在需要通信的独立容器(standalone containers)中时，通常会选择`bridge`模式。
* host：移除容器和`Docker`宿主机之间的网络隔离，并直接使用主机的网络。`host`模式仅适用于`Docker 17.06+`。
* overlay：overlay网络将多个Docker守护进程连接在一起，并使集群服务能够相互通信。您还可以使用overlay网络来实现swarm集群和独立容器之间的通信，或者不同Docker守护进程上的两个独立容器之间的通信。该策略实现了在这些容器之间进行操作系统级别路由的需求。
* macvlan：Macvlan网络允许为容器分配MAC地址，使其显示为网络上的物理设备。 Docker守护进程通过其MAC地址将流量路由到容器。对于希望直连到物理网络的传统应用程序而言，使用 macvlan模式一般是最佳选择，而不应该通过Docker宿主机的网络进行路由。
* none：对于此容器，禁用所有联网。通常与自定义网络驱动程序一起使用。none模式不适用于集群服务。

此篇细节很多很多，可以参看[手册](https://docs.docker.com/network/)

### 常用命令
* docker

```
  docker images // 镜像列表
  docker pull ubuntu // 拉取镜像
  docker search ubuntu // 搜索镜像
  docker rmi ubuntu // 删除镜像
  docker tag 860c279d2fec runoob/centos:dev // 添加标签

  docker stats --help // 状态
  docker run -it ubuntu /bin/bash // 以ubuntu镜像创建一个容器并且进入容器
  docker ps // 查看正在运行的容器
  docker start/stop/restart b750bbbcfd88 // 启动/停止/重启指定的容器
  docker run -b750bbbcfd88 --name ubuntu-test ubuntu /bin/bash // 后台运行
  docker exec -it b750bbbcfd88 /bin/bash // 进入容器
  docker rm -f 1e560fca3906 // 删除容器
```

* Dockerfile构建镜像文件
* Docker Compose统一编排镜像
* 删除

```
1.停止所有的container，这样才能够删除其中的images：
docker stop $(docker ps -a -q)

如果想要删除所有container的话再加一个指令：
docker rm $(docker ps -a -q)

2.查看当前有些什么images
docker images

3.删除images，通过image的id来指定删除谁
docker rmi <image id>

想要删除untagged images，也就是那些id为<None>的image的话可以用

docker rmi $(docker images | grep "^<none>" | awk "{print $3}")

要删除全部image的话

docker rmi $(docker images -q)
```

## 额外练习
参考5

## 参考
1. [https://en.wikipedia.org/wiki/Docker_(software)](https://en.wikipedia.org/wiki/Docker_(software))
2. [https://www.linuxidc.com/Linux/2014-09/106322.htm](https://www.linuxidc.com/Linux/2014-09/106322.htm)
3. [https://docs.docker.com/engine/docker-overview/](https://docs.docker.com/engine/docker-overview/)
4. [https://cloud.tencent.com/developer/article/1110563](https://cloud.tencent.com/developer/article/1110563)
5. [https://www.runoob.com/docker/docker-install-ubuntu.html](https://www.runoob.com/docker/docker-install-ubuntu.html)