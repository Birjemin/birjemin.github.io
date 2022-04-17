 # Kubernetes开发中的新功能
 本章对Kubernetes开发中的一些新功能进行介绍，包括对Windows 容器的支持、对GPU的支持、Pod的垂直扩缩容，并讲解Kubernetes的演进路线(Roadmap)和开发模式。

## 对Windows容器的支持
### Windows Node部署
## 对GPU的支持
## Pod的垂直扩缩容
Kubernetes在1.9版本中加入了对Pod的垂直扩缩容(简称为VPA) 支持，这一功能使用CRD的方式为Pod定义垂直扩缩容的规则，根据Pod 的运行行为来判断Pod的资源需求，从而更好地为Pod􏰀供调度支持。

- 能够在AWS、Azure和GCP上􏰀供集群节点扩缩容的 ClusterAutoScaler，已经进入GA阶段;
- 提供Pod垂直扩缩容的Vertical Pod Autoscaler，目前处于Beta阶 段;
- Addon Resizer，是Vertical Pod Autoscaler的简化版，能够根据 节点数量修改Deployment资源请求，目前处于Beta阶段。

#### 注意事项
注意事项如下。
- VPA对Pod的更新会造成Pod的重新创建和调度。
- 对于不受控制器支配的Pod，VPA仅能在其创建时􏰀供支持。
- VPA的准入控制器是一个Webhook，可能会和其他同类 Webhook存在冲突，从而导致无法正确执行。
- VPA无法和使用CPU、内存指标的HPA共用。
- VPA能够识别多数内存不足的问题，但并非全部。
- 尚未在大规模集群上测试VPA的性能。
- 如果多个VPA对象都匹配同一个Pod，则会造成不可预知的后 果。
- VPA目前不会修改limits字段的内容。

## Kubernetes的演进路线和开发模式

## 备注