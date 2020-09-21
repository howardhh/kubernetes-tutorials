<img src="pictures/black_lives_matter.png" width=800px>

# 基于 Kubernetes 的应用发布

## 回顾

### Docker
* 镜像：用来创建容器的只读模板，通常是基于另外一个镜像，并添加一些自定义的内容。
* 容器：容器是从镜像创建的运行实例，它可以被启动，开始、停止、删除、每个容器都是互相隔离的，保证安全的平台，可以把容器看做是一个简易版的linux环境（包括root用户权限、镜像空间、用户空间和网络空间等）和运行在其中的应用程序。
* 镜像仓库：镜像仓库是集中存储镜像的地方，分为公共仓库和私有仓库。
  http://11.8.38.55:9080/kubernetes/docker/


<img src="pictures/docker-architecture.svg" width=800px>
<br><br>

### 云原生
云原生是一种行为方式和设计理念，凡是能够提高云上资源利用率和应用交付效率的行为或方式都是云原生的。

云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。云原生的代表技术包括 容器、服务网格、微服务、不可变基础设施 和 声明式 API。这些技术能够构建容错性好、易于管理和便于观察的松耦合系统。结合可靠的自动化手段，云原生技术使工程师能够轻松地对系统作出频繁和可预测的重大变更。

Kubernetes 作为云原生应用的基石，相当于一个云操作系统，其重要性不言而喻。如果想构建云原生架构，建议直接从 Kubernetes 入手即可。

### [Kubernetes](https://kubernetes.io/zh/)
Kubernetes（K8s）是用于自动部署，扩展和管理容器化应用程序的开源系统。

* 使用 Docker 对应用程序包装、实例化
* 以集群的方式运行、管理跨机器的容器
* 解决Docker跨机器容器之间的通讯问题
* Kubernetes的自我修复机制使得容器集群总是运行在用户期望的状态

Kubernetes 是一个容器管理系统，其中容器在没有特殊情况下即为Docker。同时，Kubernetes 提供了一种规范，可以让你来描述集群的架构，定义服务的最终状态，帮你将系统自动地达到和维持在这个状态。

* [Pod](https://kubernetes.io/zh/docs/concepts/workloads/pods/): Kubernetes 最基本的部署调度单元。每个 Pod 由一个或多个业务容器和一个根容器（Pause容器）组成，同一个 Pod 内的容器共享网络命名空间和文件系统，容器间可以通过 localhost 通信。通常，一个Pod表示某个应用的一个实例。
* [Service](https://kubernetes.io/zh/docs/concepts/services-networking/service/): Service 定义了服务的访问入口，服务的调用者通过这个地址访问 Service 后端的 Pod 副本实例。Service 通过 Label Selector 同后端的 Pod 副本建立关系。
  
![kubernetes-cluster](./pictures/kubernetes-cluster.png)

* [Deployment](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)：副本控制器，为 Pods 和 ReplicaSets 提供声明式的更新能力。保证后端 Pod 副本的数量，也就是保证服务的伸缩性。
  
![deployment-rs-pod](./pictures/deployment-rs-pod.png)
  
  * 创建 Deployment 以将 ReplicaSet 上线。 ReplicaSet 在后台创建 Pods。 检查 ReplicaSet 的上线状态，查看其是否成功。
  * 通过更新 Deployment 的 PodTemplateSpec，声明 Pod 的新状态 。 新的 ReplicaSet 会被创建，Deployment 以受控速率将 Pod 从旧 ReplicaSet 迁移到新 ReplicaSet。 每个新的 ReplicaSet 都会更新 Deployment 的修订版本。
  * 如果 Deployment 的当前状态不稳定，回滚到较早的 Deployment 版本。 每次回滚都会更新 Deployment 的修订版本。
  * 扩大 Deployment 规模以承担更多负载。
  * 暂停 Deployment 以应用对 PodTemplateSpec 所作的多项修改， 然后恢复其执行以启动新的上线版本。
  * 使用 Deployment 状态 来判定上线过程是否出现停滞。
  * 清理较旧的不再需要的 ReplicaSet。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
<br><br>

<!-- 开发只需关心deployment，不需要关心其他 -->

## 应用发布

### 停机发布( **Recreate** )
或者叫做直接替换部署，一般应用于服务器需求较少的业务，优点是降低资源成本，部署方便；缺点是业务需要停止服务，如某些证券公司在不开盘的时候都会停服维护。

Kubernetes提供的两种策略之一。停止所有运行的旧版本实例，再重新创建新版本实例。
```yaml
spec:
  strategy:
    type: Recreate
```
<br><br>

### 滚动发布( **RollingUpdate** )
在升级过程中，并不一下子启动所有新版本，是先启动一台新版本，再停止一台老版本，然后再启动一台新版本，再停止一台老版本，直到升级完成。

优点是可以保持 0 downtime ，实现了不停服更新，比较节省资源。缺点是在一段时间内，线上运行两个不同版本，出现问题时，无法保证一个稳定的线上环境。

<img src="pictures/rollingupdate.png" width=800px>

**RollingUpdate** 是 Kubernetes 默认的发布策略，更新时，Deployment 会创建一个新的 ReplicaSet，并逐渐在新 ReplicaSet 中添加新版本 Pod 实例，减少旧 ReplicaSet 中的 旧版本 Pod 实例，直到完成所有副本完成升级。

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2       # 最大超出期望 Pod 个数上限，默认为25%
      maxUnavailable: 0 # 最大不可用 Pod 个数上限，默认为25%
```
<br><br>

### 蓝绿发布
蓝色环境指当前正在使用的生产环境，绿色环境指将要发布新版本的环境（热备环境），通过在两个生产环境中切换流量来实现应用版本的更新。应用更新时，蓝色环境仍然对外提供服务，绿色环境升级应用版本，测试通过后，通过负载均衡变更，将流量指向绿色环境。

<img src="pictures/blue_and_green.png" width=800px>

优点是操作简单，可以在绿色环境下充分测试，回滚速度快。缺点是成本高。

蓝绿发布与 Kubernetes 中 Deployment 的更新策略无关，可以依靠更新 Service 切换流量到不同版本 Pod 实例来实现，或使用第三方部署插件实现自动化。适合使用 Kubernetes 实现。
<br><br>

### 灰度发布
灰度发布也叫金丝雀发布，起源是，矿井工人发现，金丝雀对瓦斯气体很敏感，矿工会在下井之前，先放一只金丝雀到井中，如果金丝雀不叫了，就代表瓦斯浓度高。

部署新版本后，通过负载均衡控制流量将新版本提供给少量用户（金丝雀），在确认新版本没问题之后实施逐步更新或全部更新。风险介于滚动发布与蓝绿发布之间。

优点是降低风险，便于发现问题和快速回滚。缺点是发布速度较慢。

<img src="pictures/grey_deploy.png" width=800px>

灰度发布在 Kubernetes 中可以借助 Service 和副本数量简单实现，或使用 HAProxy、Istio 等工具实现更精确的流量控制。
<br><br>

### 版本更新与回滚
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
# or
kubectl edit deployment.v1.apps/nginx-deployment
```

```bash
kubectl rollout undo deployment.v1.apps/nginx-deployment
# or
kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
```
<br><br>

## CI/CD
### [GitOps](https://zhuanlan.zhihu.com/p/43002417)
GitOps是一种实现持续交付的模型，核心思想是将应用系统的声明性基础架构和应用程序存放在Git的版本控制库中。

将Git作为交付流水线的核心，每个开发人员都可以提交拉取请求（Pull Request）并使用Gi​​t来加速和简化Kubernetes的应用程序部署和运维任务。通过使用像Git这样的简单工具，开发人员可以更高效地将注意力集中在创建新功能而不是运维相关任务上（例如，应用系统安装、配置、迁移等）。

#### 应用场景
* 不可变基础设施：GitOps是在具体Kubernetes的应用实践中出现的，GitOps需要依托于“不可变基础架构”才能发挥其作用。在一定程度上说，“不可变基础架构”为GitOps的出现创造了必要的条件，反过来GitOps应用Kubernetes的容器编排能力，能够迅速的使用镜像搭建出应用系统所需的组件。
* 声明式容器编排：借助Kubermetes的声明性特点，应用系统的整个配置文件集可以在Git库中进行版本控制。通过使用Git库，应用程序更容易部署到Kubernetes中，以及进行版本回滚。更重要的是，当灾难发生时，群集的基础架构可以从Git库中可靠且快速地恢复。

#### 基本原则
* 任何能够被描述的内容都必须存储在Git库中
* 不应直接使用kubectl
* 调用Kubernetes 的API的接口或者控制器应该遵循 Operator 模式


<img src="pictures/gitops-pipeline.png" width=800px>
<br><br>

### Suagr-CI
![sugar-ci](./pictures/sugar-cicd.png)

### [GitLab](https://docs.gitlab.com/ee/ci/yaml/)
![gitlab-ci](./pictures/gitlab-ci.png)

* Gitlab Runner：在 Kubernetes 集群中安装 runner 与 Gitlab 服务端通信
* Kubernetes Executor：runner 的插件，用于与 Kubernetes 集群通信
* .gitlab-ci.yml 位于 git 项目根目录的一个文件，记录了 pipeline 的阶段和执行规则

  
### [Spinnaker](https://spinnaker.io/)
Netflix 在2015年开源的持续交付平台。
* 支持多云（AWS/GCP/Azure/kubernetes/openstack），不支持国内云平台，差评
* CI/CD，策略发布，监控集成
* UI友好，pipeline功能完善，支持金丝雀发布
* 配置复杂，上手难度大，不支持 GitOps
  
### [Argo CD](https://argoproj.github.io/argo-cd/)
Argo CD是一个为Kubernetes而生的、遵循声明式GitOps理念的持续部署工具。
* 应用定义、配置和环境信息是声明式的，并且可以进行版本控制；
* 应用部署和生命周期管理是全自动化的，是可审计的，清晰易懂；
* Argo CD是一个独立的部署工具，支持对多个环境、多个Kubernetes集群上的应用进行统一部署和管理。

### [Tekton](https://tekton.dev)
Tekton 是一个功能强大且灵活的 Kubernetes **原生**框架，用于创建 CI/CD 系统。


### JenkinsX
Jenkins X是基于Kubernetes的持续集成、持续部署平台。

