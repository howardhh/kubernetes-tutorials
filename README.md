# kubernetes-tutorials
学习Kubernetes已有一段时间，又赶上领导让准备下做个培训，所以将这段时间我个人学习和理解的一些内容记录在此，其中必然诸多纰漏，还望轻喷。

## 介绍
**Production-Grade Container Orchestration**

[Kubernetes](https://kubernetes.io/)（K8s）是用于自动部署，扩展和管理容器化应用程序的开源系统。部分重要特性如下：

* Service discovery and load balancing：无需修改您的应用程序即可使用陌生的服务发现机制。Kubernetes 为容器提供了自己的 IP 地址和一个 DNS 名称，并且可以在它们之间实现负载平衡。
* Self-healing: 重新启动失败的容器，在节点死亡时替换并重新调度容器，杀死不响应用户定义的健康检查的容器，并且在它们准备好服务之前不会将它们公布给客户端。
* Automatic bin packing：根据资源需求和其他约束自动放置容器，同时不会牺牲可用性，将任务关键工作负载和尽力服务工作负载进行混合放置，以提高资源利用率并节省更多资源。
* Automated rollouts and rollbacks：Kubernetes 会逐步推出针对应用或其配置的更改，确保在监视应用程序运行状况的同时，不会终止所有实例。如果出现问题，Kubernetes 会为您回滚更改。充分利用不断成长的部署解决方案生态系统。
* Horizontal scaling：使用一个简单的命令、一个UI或基于CPU使用情况自动对应用程序进行伸缩。

简单来说，Kubernetes是一个容器管理系统，其中容器在没有特殊情况下即为[Docker](https://www.docker.com/)。同时，Kubernetes提供了一种规范，可以让你来描述集群的架构，定义服务的最终状态，帮你将系统自动地达到和维持在这个状态。

## 基本概念及组件
### Cluster - 集群
集群是一组**节点**，可以是物理机或虚拟机，<del>甚至可以是容器</del>。节点的角色分为**Master**和**Node**，Master负责整个集群的管理和控制，Node负责运行容器应用。开发测试环境中Master和Node可以部署在相同节点，生产环境则应分开部署。
![kubernetes-cluster](./pictures/kubernetes-cluster.png)

### Container - 容器
容器是一个Kubernetes集群中的最小单元，在上面运行着具体的应用进程。
### Pod












