# Trouble Shooting指导
为了跟踪和发现在Kubernetes集群中运行的容器应用出现的问题，我们常用如下查错方法。
- 查看Kubernetes对象的当前运行时信息，特别是与对象关联的 Event事件。这些事件记录了相关主题、发生时间、最近发生时间、发生次数及事件原因等，对排查故障非常有价值。此外，通过查看对象的运行时数据，我们还可以发现参数错误、关联错误、状态异常等明显问 题。由于在Kubernetes中多种对象相互关联，因此这一步可能会涉及多 个相关对象的排查问题。
- 对于服务、容器方面的问题，可能需要深入容器内部进行故 障诊断，此时可以通过查看容器的运行日志来定位具体问题。
- 对于某些复杂问题，例如Pod调度这种全局性的问题，可能需 要结合集群中每个节点上的Kubernetes服务日志来排查。比如搜集 Master上的kube-apiserver、kube-schedule、kube-controler-manager服务日 志，以及各个Node上的kubelet、kube-proxy服务日志，通过综合判断各 种信息，就能找到问题的成因并解决问题。

## 查看系统Event
在Kubernetes集群中创建Pod后，我们可以通过kubectl get pods命令 查看Pod列表，但通过该命令显示的信息有限。Kubernetes􏰀供了kubectl describe pod命令来查看一个Pod的详细信息，例如:
```
kubectl describe pod <pod-name>
kubectl describe service <svc-name>
```
## 查看容器日志
```
kubectl logs <pod-name>
```
## 查看Kubernetes服务日志
如果在Linux系统上安装Kubernetes，并且使用systemd系统管理 Kubernetes服务，那么systemd的journal系统会接管服务程序的输出日 志。在这种环境中，可以通过使用systemd status或journalctl工具来查看 系统服务的日志。
```
systemctl status kube-contoller-manager -l
journalctl -u kube-controller-manager
```
如果某个Kubernetes对象存在问题，则可以用这个对象的名字作为 关键字搜索Kubernetes的日志来发现和解决问题。在大多数情况下，我 们遇到的主要是与Pod对象相关的问题，比如无法创建Pod、Pod启动后 就停止或者Pod副本无法增加，等等。此时，可以先确定Pod在哪个节点 上，然后登录这个节点，从kubelet的日志中查询该Pod的完整日志，然 后进行问题排查。对于与Pod扩容相关或者与RC相关的问题，则很可能 在kube-controller-manager及kube-scheduler的日志中找出问题的关键点。
另外，kube-proxy经常被我们忽视，因为即使它意外停止，Pod的状 态也是正常的，但会导致某些服务访问异常。这些错误通常与每个节点 上的kube-proxy服务有着密切的关系。遇到这些问题时，首先要排查 kube-proxy服务的日志，同时排查防火墙服务，要特别留意在防火墙中 是否有人为添加的可疑规则。

## 常见问题
### 由于无法下载pause镜像导致Pod一直处于Pending状态
### Pod创建成功，但Restarts数量持续增加
这通常是因为容器的启动命令不能保持在前台运行。

### 通过服务名无法访问服务
在Kubernetes集群中应尽量使用服务名访问正在运行的微服务，但有时会访问失败。由于服务涉及服务名的DNS域名解析、kube-proxy组 件的负载分发、后端Pod列表的状态等，所以可通过以下几方面排查问 题。
1. 查看Service的后端Endpoint是否正常
2. 查看Service的名称能否被正确解析为ClusterIP地址
3. 查看kube-proxy的转发规则是否正确

## 备注