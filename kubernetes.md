# Kubernetes

## 核心组件

管控面组件

- etcd：高可用的键值存储数据库，保存所有的集群数据
- apiserver：资源操作的唯一入口，提供认证、授权、访问控制、API注册和发现等机制；
- controller manager：负责管理集群各种资源，确保处于期望的状态，比如故障检测、自动扩展、滚动更新等；
- scheduler：负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；

工作节点的组件

- kubelet：确保容器处于运行状态且健康
- kube-proxy：网络代理，维护节点上的网络规则，为Service提供服务发现和负载均衡；

### Kube-proxy的发展

- 第一代：Kube-proxy进程是一个真实的TCP/UDP代理，负责Service到Pod的访问流量
- 第二代：将iptables作为kube-proxy的默认模式，iptables模式工作在内核态，不经过用户态中转，性能更强
- 第三代：集群中service和Pod大量增加后，每个node上iptables的规则会急速膨胀，导致网络性能下降，于是引入了IPVS，专用于高性能负载均衡，使用哈希表

## 工作负载

### deployment和statefulSet的区别

1. **有状态和无状态**：Deployment 适用于无状态服务，它可以无差别地对 Pod 进行扩展、更新和回滚。而 StatefulSet 适用于有状态服务，它能保证 Pod 的部署和扩展顺序，并在重新调度时保持 Pod 的网络标识和存储。
2. **Pod 的标识**：Deployment 创建的 Pod 没有稳定的网络标识，每次调度都可能改变。相反，StatefulSet 为每个 Pod 分配一个稳定、唯一的序号，Pod 的名称和网络标识在整个生命周期内保持不变。
3. **更新策略**：在 Deployment 中，新的 Pod 可以在任何顺序中创建和删除，而在 StatefulSet 中，最高编号的 Pod 会首先被删除，当新 Pod 创建时，最低编号的 Pod 首先被创建
4. **存储**：Deployment 通常用于无状态应用，一般不需要持久化存储。但如果需要，可以使用 Persistent Volumes。而 StatefulSet 支持每个 Pod 有自己的持久化存储，即每个 Pod 可以绑定一个单独的 Persistent Volume。

### deployment创建更新的过程

更新：滚动升级，最多不可用和允许超过数量为所需副本数量的25%

## Service

### service和ingress 的区别

Service主要用于在集群内部的服务发现和负载均衡，而Ingress则用于将外部流量路由到Service和Pod上

## Scheduler

Scheduling Framework

### 调度流程

承上：负责接收Controller Manager创建的新Pod

启下：调度到Node上后，由Node上的kubelet服务接管后续工作

## CRD的性能瓶颈

1. **API服务器的负载：** CRD是通过Kubernetes API服务器进行管理的，过多的CRD对象可能会导致API服务器的负载过高。
2. **Etcd性能：** Kubernetes所有的状态信息，包括CRD的信息，都存储在Etcd中。如果CRD的数量非常多，或者CRD对象的大小非常大，可能会对Etcd造成压力，影响其性能。
3. **控制器的性能：** 如果你为CRD创建了自定义控制器，那么控制器代码的性能也会影响整个系统。例如，控制器的逻辑可能会导致过多的API调用，或者处理单个CRD对象的时间过长，这都可能成为性能瓶颈。
4. **网络性能：** 如果CRD关联的操作涉及到网络请求，比如调用外部API，那么网络延迟和带宽也可能成为性能瓶颈。

## 网络

### flannel和calico的区别

flannel：是一个简单的容器网络方案，基于一个名为 `flanneld`的网桥，以及绑定在网桥上的 `vethpair`作为节点上的容器网络架构，在 `flannel`方案下，每个k8s node都支持一段 `cidr`

calico：`calico`是一个基于 `bgp`协议实现的纯路由网络方案，calico保证所有容器的数据流量都是通过IP路由的方式完成互联互通

Flannel注重易用性和简单性，而Calico提供了更多的网络策略和更好的性能

## 存储

## Empty dir和PV区别
