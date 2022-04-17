# Kubernetes开发指南
## REST简述
设计准则：
- 网络上的所有事物都被抽象为资源(Resource)。
- 每个资源都对应唯一的资源标识符(Resource Identifier)。
- 通过通用的连接器接口(Generic Connector Interface)对资源进行操作。
- 对资源的各种操作都不会改变资源标识符。
- 所有操作都是无状态的(Stateless)。

## Kubernetes API详解
### Kubernetes API概述
Kubernetes API是集群系统中的重要组成部分，Kubernetes中各种资 源(对象)的数据都通过该API接口被提交到后端的持久化存储 (etcd)中，Kubernetes集群中的各部件之间通过该API接口实现解耦合，同时Kubernetes集群中一个重要且便捷的管理工具kubectl也是通过 访问该API接口实现其强大的管理功能的。Kubernetes API中的资源对象 都拥有通用的元数据，资源对象也可能存在嵌套现象，比如在一个Pod 里面嵌套多个Container。创建一个API对象是指通过API调用创建一条有 意义的记录，该记录一旦被创建，Kubernetes就将确保对应的资源对象 会被自动创建并托管维护。

在Kubernetes API中，一个API的顶层(Top Level)元素 由kind、apiVersion、metadata、spec和status这5部分组成
- kind
  - 对象（objects）
  - 列表（list）
  - 简单类别（simple）
- apiVersion
- Metadata
  - namespace
  - name
  - uid
  - labels
  - annotations
  - resourceVersion
  - creationTimestamp
  - deletionTimestamp
  - selfLink
- spec
- Status
  - phase
  - condition

### Kubernetes API版本的演进策略
API的版本号通常用于描述API的成熟阶段，例如: 
- v1表示GA稳定版本;
- v1beta3表示Beta版本(预发布版本);
- v1alpha1表示Alpha版本(实验性的版本)。

当某个API的实现达到一个新的GA稳定版本时(如v2)，旧的GA 版本(如v1)和Beta版本(例如v2beta1)将逐渐被废弃，Kubernetes建议废弃的时间如下。
- 对于旧的GA版本(如v1)，Kubernetes建议废弃的时间应不少于12个月或3个大版本Release的时间，选择最长的时间。
- 对旧的Beta版本(如v2beta1)，Kubernetes建议废弃的时间应不少于9个月或3个大版本Release的时间，选择最长的时间。
- 对旧的Alpha版本，则无须等待，可以直接废弃。


### API Groups（API组）
### API REST的方法说明
- GET /<资源名的复数格式>
- POST /<资源名的复数格式>
- GET /<资源名复数格式>/<名称>
- DELETE /<资源名复数格式>/<名称>
- PUT /<资源名复数格式>/<名称>
- PATCH /<资源名复数格式>/<名称>

### API Server响应说明

## 使用Java程序访问Kubernetes API

## Kubernetes API的扩展
### 使用CRD扩展API资源
### 使用API聚合机制扩展API资源

## 参考

