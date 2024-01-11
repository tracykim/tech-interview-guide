# Prometheus

## prometheus架构

![Prometheus architecture](./images/z6cqx.png)

### Prometheus Server

Prometheus Server是Prometheus组件中的核心部分，负责实现对监控数据的**获取，存储以及查询**。

- 获取数据(Retrival)。Prometheus Server可以通过静态配置管理监控目标，也可以配合使用Service Discovery的方式动态管理监控目标，并从这些监控目标中获取数据。
- 存储数据(Storage)。其次Prometheus Server需要对采集到的监控数据进行存储，Prometheus Server本身就是一个时序数据库，将采集到的监控数据按照时间序列的方式存储在本地磁盘当中。
- 查询数据。最后Prometheus Server对外提供了自定义的PromQL语言，实现对数据的查询以及分析。

### Exporter

Exporter将监控数据采集的端点通过HTTP服务的形式暴露给Prometheus Server，Prometheus Server通过访问该Exporter提供的Endpoint端点，即可获取到需要采集的监控数据。

一般来说可以将Exporter分为2类：

- 直接采集：这一类Exporter直接内置了对Prometheus监控的支持，比如cAdvisor，Kubernetes，Etcd，Gokit等，都直接内置了用于向Prometheus暴露监控数据的端点。
- 间接采集：间接采集，原有监控目标并不直接支持Prometheus，因此我们需要通过Prometheus提供的Client Library编写该监控目标的监控采集程序。例如： Mysql Exporter，JMX Exporter，Consul Exporter等。

### AlertManager

AlertManager即Prometheus体系中的告警处理中心。在Prometheus Server中支持基于PromQL创建告警规则，如果满足PromQL定义的规则，则会产生一条告警，而告警的后续处理流程则由AlertManager进行管理。在AlertManager中我们可以与邮件，Slack等等内置的通知方式进行集成，也可以通过Webhook自定义告警处理方式。

### PushGateway

由于Prometheus数据采集基于Pull模型进行设计，因此在网络环境的配置上必须要让Prometheus Server能够直接与Exporter进行通信。 当这种网络需求无法直接满足时，就可以利用PushGateway来进行中转。可以通过PushGateway将内部网络的监控数据主动Push到Gateway当中。而Prometheus Server则可以采用同样Pull的方式从PushGateway中获取到监控数据。

### Service Discovery

在Kubernetes中，pod会随时在创建和被销毁。这种按需的资源使用方式对于监控系统而言就意味着没有了一个固定的监控目标，所有的监控对象(基础设施、应用、服务)都在动态的变化。对于Prometheus这一类基于Pull模式的监控系统，显然也无法继续使用的static_configs的方式静态的定义监控目标。而对于Prometheus而言其解决方案就是引入一个中间的代理人（服务注册中心），这个代理人掌握着当前所有监控目标的访问信息，Prometheus只需要向这个代理人询问有哪些监控目标控即可， 这种模式被称为服务发现。

Prometheus通过使用平台提供的API就可以找到所有需要监控的云主机。在Kubernetes这类容器管理平台中，Kubernetes掌握并管理着所有的容器以及服务信息，那此时Prometheus只需要与Kubernetes打交道就可以找到所有需要监控的容器以及服务对象。Prometheus还可以直接与一些开源的服务发现工具进行集成，例如在微服务架构的应用程序中，经常会使用到例如Consul这样的服务发现注册软件，Promethues也可以与其集成从而动态的发现需要监控的应用服务实例。

## 监控组件

- node_exporter 提供了节点级别的指标，帮助你了解每个节点的健康状况和性能。
- cAdvisor 提供了容器级别的指标，让你了解每个容器的运行状况。
- kube-state-metrics 提供了 Kubernetes 对象的状态指标，帮助你了解 Kubernetes 集群的状态。
  - **Node**：节点的总数、状态（例如，是否处于就绪状态）、CPU 和内存容量等。
  - **Pod**：各个命名空间中的 Pod 数量、Pod 的状态（如 Pending、Running、Succeeded、Failed）、Pod 中的容器数量等。
  - **Deployment**：部署的总数、期望的副本数、当前的副本数、可用的副本数等。
  - **Service**：服务的总数、服务的类型（如 ClusterIP、NodePort、LoadBalancer）等。
  - **Job**：Job 的总数、已完成的 Job 数量、失败的 Job 数量等。
  - **Namespace**：命名空间的总数、各个状态（如 Active、Terminating）的命名空间数量等

# CICD

## Devops

DevOps是一种软件开发方法论，它强调开发（Dev）和运维（Ops）两个部分的紧密合作以提高系统的交付效率和产品质量。目标是更短的发布周期，更高的部署频率，以及能更快地修复问题

## CI

CI（Continuous Integration）是持续集成，开发人员在开发过程中频繁地将代码集成到主分支。每次集成都通过自动化的构建（包括编译、发布、自动化测试）来验证，以便尽早发现集成错误

## CD

持续交付（Continuous Delivery），开发人员对软件的更改在构建后尽快交付给用户。目标是构建、测试和发布软件更快、更频繁。每一个通过CI测试的更改都可以被视为准备好交付的候选版本。

持续部署（Continuous Deployment）是持续交付的下一步，不仅将更改交付给用户进行测试，而且在通过测试后将这些更改自动部署到生产环境。这意味着每一次更改后的应用，只要通过了所有的自动化测试，都会自动部署到生产环境

## 敏捷开发

敏捷开发是一种软件开发方法，它强调灵活性和客户合作，以应对需求和解决方案的快速变化。这种方法的核心是通过迭代和增量开发来适应这种变化，每次迭代都会产出一个可工作的产品增量。敏捷开发注重提高客户满意度，欢迎并适应需求变更，频繁地交付可工作的软件
