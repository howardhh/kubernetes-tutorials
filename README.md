# kubernetes-tutorials
学习和使用 Kubernetes 已有一段时间，又赶上领导让准备下做个培训，所以将这段时间我个人学习和理解的一些内容记录在此。

## 介绍
**Production-Grade Container Orchestration**

[Kubernetes](https://kubernetes.io/)（K8s）是用于自动部署，扩展和管理容器化应用程序的开源系统。部分重要特性如下：

* Service discovery and load balancing：无需修改您的应用程序即可使用陌生的服务发现机制。Kubernetes 为容器提供了自己的 IP 地址和一个 DNS 名称，并且可以在它们之间实现负载平衡。
* Self-healing: 重新启动失败的容器，在节点死亡时替换并重新调度容器，杀死不响应用户定义的健康检查的容器，并且在它们准备好服务之前不会将它们公布给客户端。
* Automatic bin packing：根据资源需求和其他约束自动放置容器，同时不会牺牲可用性，将任务关键工作负载和尽力服务工作负载进行混合放置，以提高资源利用率并节省更多资源。
* Automated rollouts and rollbacks：Kubernetes 会逐步推出针对应用或其配置的更改，确保在监视应用程序运行状况的同时，不会终止所有实例。如果出现问题，Kubernetes 会为您回滚更改。充分利用不断成长的部署解决方案生态系统。
* Horizontal scaling：使用一个简单的命令、一个UI或基于CPU使用情况自动对应用程序进行伸缩。

简单来说，Kubernetes 是一个容器管理系统，其中容器在没有特殊情况下即为[Docker](https://www.docker.com/)。同时，Kubernetes 提供了一种规范，可以让你来描述集群的架构，定义服务的最终状态，帮你将系统自动地达到和维持在这个状态。

## 基本概念及组件
### Cluster - 集群
集群是一组**节点**，可以是物理机或虚拟机，<del>甚至可以是容器</del>。节点的角色分为**Master**和**Node**，Master 负责整个集群的管理和控制，Node 负责运行容器应用。开发测试环境中 Master 和 Node 可以部署在相同节点，生产环境则应分开部署。
![kubernetes-cluster](./pictures/kubernetes-cluster.png)

### Container - 容器
容器是Kubernetes集群中的低级别组件，在上面运行着具体的应用进程。相关内容请参考：[Docker容器介绍](./components/docker.md)

### Pod
Pod 是 Kubernetes 最基本的部署调度单元。每个 Pod 由一个或多个业务容器和一个根容器（Pause容器）组成，同一个 Pod 内的容器共享网络命名空间和文件系统，容器间可以通过 localhost 通信。通常，一个Pod表示某个应用的一个实例。

[More...](./components/pod.md)

### Service
Service 是 Kubernetes 最重要的资源对象。Kubernetes 中的 Service 对象可以对应微服务架构中的微服务。Service 定义了服务的访问入口，服务的调用者通过这个地址访问 Service 后端的 Pod 副本实例。Service 通过 Label Selector 同后端的 Pod 副本建立关系，Deployment 保证后端Pod 副本的数量，也就是保证服务的伸缩性。

[More...](/.components/service.md)

### Node
Node 也可称为 Worker，是 Kubernetes 集群中的工作节点，主要包含如下组件：

* kubelet：负责 Pod 的创建、启动、监控、重启、销毁等工作，同时与 Master 节点协作，实现集群管理的基本功能。
* kube-proxy：实现 Kubernetes Service之间的通信和负载均衡

### Master
Master 是 Kubernetes 集群中的控制节点，主要包含如下组件：

* kube-apiserver：集群控制的入口，提供 HTTP REST 服务
* kube-controller-manager：Kubernetes 集群中所有资源对象的自动化控制中心
* kube-scheduler：负责 Pod 的调度

[More...](./components/node.md)

### Etcd
Etcd是一个分布式的 key-value 存储，作为保存整个集群状态的数据库。

## 组件间通信
一个最典型的创建Pod的流程如下：

![k8s-pod-process](./pictures/k8s-pod-process.png)

* 用户通过 kube-apiserver 提供的 REST API 创建一个 Pod
* kube-apiserver 将数据写入 etcd
* kube-scheduler 检测到未绑定 Node 的 Pod，开始调度并更新 Pod 的 Node 绑定
* kubelet 检测到有新的 Pod 调度过来，通过 container runtime 运行该 Pod
* kubelet 通过 container runtime 取到 Pod 状态，并更新到 apiserver 中
















