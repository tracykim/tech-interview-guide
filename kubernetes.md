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

### 1、Kube-proxy

#### Kube-proxy的发展

- 第一代：Kube-proxy进程是一个真实的TCP/UDP代理，负责Service到Pod的访问流量
- 第二代：将iptables作为kube-proxy的默认模式，iptables模式工作在内核态，不经过用户态中转，性能更强
- 第三代：集群中service和Pod大量增加后，每个node上iptables的规则会急速膨胀，导致网络性能下降，于是引入了IPVS，专用于高性能负载均衡，使用哈希表

#### kube-proxy的作用

 kube-proxy负责为Service提供cluster内部的服务发现和负载均衡，它运行在每个Node计算节点上，负责Pod网络代理, 它会定时从etcd服务获取到service信息来做相应的策略，维护网络规则和四层负载均衡工作

### 2、Scheduler

Scheduling Framework

### 调度流程

承上：负责接收Controller Manager创建的新Pod

启下：调度到Node上后，由Node上的kubelet服务接管后续工作

### 3、controller-manager

#### informer

Informer是 Kubernetes 控制器（controller）中的模块，是控制器调谐循环（reconcile loop）与 Kubernetes API-Server 挂接的桥梁，我们通过 API-Server 增删改某个 Kubernetes API 对象，该资源对应的控制器中的 Informer 会立即感知到这个事件并作出调谐。informer会周期性的执行List（resync）操作，将所有的资源存放在Informer Store中

![img](./images/informer.jpg)

Informer架构设计的核心组件

- Reflector：用于List&Watch指定的Kubernetes资源，当监控的资源发生变化时触发响应的变更事件（例如Added事件、Updated事件和Deleted事件），并将其资源对象存放到本地缓存DeltaFIFO Queue。
- DeltaFIFO Queue：用来存储变化的kubernetes资源对象
- Indexer：将增量中的 Kubernetes 资源对象保存到本地缓存中，并为其创建索引，这份缓存与 etcd 中的数据是完全一致的。控制器只从本地缓存通过索引读取数据，这样做减小了 apiserver 和 etcd 的压力

## 工作负载

### deployment和statefulSet的区别

1. **有状态和无状态**：Deployment 适用于无状态服务，它可以无差别地对 Pod 进行扩展、更新和回滚。而 StatefulSet 适用于有状态服务，它能保证 Pod 的部署和扩展顺序，并在重新调度时保持 Pod 的网络标识和存储。
2. **Pod 的标识**：Deployment 创建的 Pod 没有稳定的网络标识，每次调度都可能改变。相反，StatefulSet 为每个 Pod 分配一个稳定、唯一的序号，Pod 的名称和网络标识在整个生命周期内保持不变。
3. **更新策略**：在 Deployment 中，新的 Pod 可以在任何顺序中创建和删除，而在 StatefulSet 中，最高编号的 Pod 会首先被删除，当新 Pod 创建时，最低编号的 Pod 首先被创建
4. **存储**：Deployment 通常用于无状态应用，一般不需要持久化存储。但如果需要，可以使用 Persistent Volumes。而 StatefulSet 支持每个 Pod 有自己的持久化存储，即每个 Pod 可以绑定一个单独的 Persistent Volume。

### deployment创建更新的过程

更新：默认是滚动升级

1. 创建新的RS，然后根据新的镜像运行新的Pod。
2. 删除旧的Pod，启动新的Pod，当新Pod就绪后，继续删除旧Pod，启动新Pod。
3. 持续第二步过程，一直到所有Pod都被更新成功。

最多不可用和允许超过数量为所需副本数量的25%

## Pod

### Pod创建过程

1. 用户通过kubectl或Deployment 控制器提交 Pod 创建事件 给 API Server
2. API Server 收到请求，将Pod 信息存入 etcd 中，待写入操作执行完成后，API Server 返回确认信息给客户端（此时Pod的状态为Pending）
3. scheduler watch到API Server 创建了新的 Pod，但尚未绑定至任何工作节点；scheduler将根据Pod 对象的spec信息（如计算资源，nodeSelector等），以及工作节点当前状态来挑选工作节点，并将调度结果信息返回给 API Server；
4. API Server将调度结果信息更新至 etcd 数据库中，而且 API Server 也开始反映此Pod 对象的调度结果（保证kube-scheduler不会重复调取该任务）
5. 目标节点上的 kubelet watch  API Server，发现有新的 Pod 调度到了自己的节点上，kubelet 在当前节点上调用容器运行时（Docker、containerd）来启动容器，并将容器的结果状态（镜像拉取状态，容器启动状态）返回给 API Server；至少有一个容器启动则Pod的状态为Running
6. API Server将 Pod 状态信息存入 etcd 数据库中；在etcd 确认写入操作成功完成后，API Server 将确认信息发送给相关节点的 kubelet
7. 将该 Pod 的 IP 地址和端口信息添加到相关 Service 的 Endpoint 对象中

### Pod删除过程

1. 用户通过命令或者是控制器向API Server提交删除Pod请求；
2. API Server 接收删除请求更新etcd
   1. 设置.metadata.deletionTimestamp
   2. 若有.metadata.finalizers，则等finalizers完成才删

3. Pod被标记为 Terminating （逻辑状态）
4. Endpoint Controller更新 Service
   1. 端点控制器（Endpoint Controller）监控 API Server 中 Pod 的状态变化
   2. 当 Pod 被标记为 `Terminating` 时，端点控制器会将该 Pod 从所有相关的 Service 的端点列表（Endpoints）中移除
   3. 端点控制器更新 Service 的端点列表，并将这些更新写入 etcd
5. 目标节点上的 kubelet  watch  API Server，发现 Pod 被标记为 Terminating 状态，开始进行清理工作，终止 Pod 中运行的所有容器
   1. kubelet 发送SIGTREM信号给所有容器，通知优雅退出。容器收到信号后，可以执行预定义的清理操作（如处理结束请求、保存状态等）。
   2. kubelet 等待所有容器优雅退出。如果配置了 `terminationGracePeriodSeconds`，kubelet 会等待指定的时间。
   3. 如果容器在该时间内未能终止，kubelet 会发送SIGKILL信息强制终止
6. 所有容器成功终止后，kubelet 将更新 Pod 的状态，并通知 API Server。
7. API Server 将 Pod 对象从 etcd 中删除

## Service

Service 是一组pod的服务抽象，主要用于为 Pod 提供一个固定、统一的访问接口及负载均衡的能力

### 服务发现

Kubernetes 支持两种主要的服务发现模式：环境变量和 DNS。

- 环境变量：当Pod启动时，Kubernetes会为每个Service在Pod的环境中设置一组环境变量。这些环境变量包含Service的集群IP和端口信息。这种方式必须在客户端 Pod 出现之前创建该 Service。 否则，这些客户端 Pod 中将不会出现对应的环境变量
- DNS：CoreDNS提供了动态服务发现功能。每个Service都会在DNS中注册一个域名，格式通常为 `<service-name>.<namespace>.svc.cluster.local`。Pod内部的应用程序可以通过该域名解析到对应的Service

### 负载均衡

Kubernetes的负载均衡通过**kube-proxy**实现，使用**iptables**或**ipvs**规则将流量均匀地分发到Service后端的Pod。流量根据轮询算法分配到多个Pod，确保高效负载均衡

主要的负载均衡算法包括：

- 轮询（Round Robin）：将流量均匀地分配到所有Pod上。
- 加权轮询（Weighted Round Robin）：给Pod分配不同的权重，权重大的Pod会收到更多的流量。
- 最少连接（Least Connections）：流量优先分配给当前连接数最少的Pod。

### service和ingress 的区别

- Service主要用于在集群内部的服务发现和负载均衡；
- Ingress则用于将外部流量路由到Service上，提供了基于域名和路径的路由规则

## client-go

- RESTClient：最基础的client
- clientSet：与k8s资源对象交互最常用的client，不包括CRD
- DynamicClient：支持CRD
- DiscoveryClient：用于发现apiserver支持的Group/Version/Resource信息

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

- emptyDir：适用于临时存储，数据与 Pod 生命周期绑定，Pod 删除后数据丢失。
- PV：适用于持久性存储，数据独立于 Pod 生命周期，提供更灵活和持久的存储解决方案。
