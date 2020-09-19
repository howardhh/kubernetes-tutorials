<!--
 * @Author: your name
 * @Date: 2020-09-13 08:48:16
 * @LastEditTime: 2020-09-19 12:02:19
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /kubernetes-tutorials/k8smore.md
-->
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

### Kubernetes
* Pod: Kubernetes 最基本的部署调度单元。每个 Pod 由一个或多个业务容器和一个根容器（Pause容器）组成，同一个 Pod 内的容器共享网络命名空间和文件系统，容器间可以通过 localhost 通信。通常，一个Pod表示某个应用的一个实例。
* Service: Service 定义了服务的访问入口，服务的调用者通过这个地址访问 Service 后端的 Pod 副本实例。Service 通过 Label Selector 同后端的 Pod 副本建立关系。
  
![kubernetes-cluster](./pictures/kubernetes-cluster.png)

* **Deployment**：副本控制器，为 Pods 和 ReplicaSets 提供声明式的更新能力。保证后端Pod 副本的数量，也就是保证服务的伸缩性。
  https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/
  
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

## 应用发布

### 停机发布( **Recreate** )
或者叫做直接替换部署，一般应用于服务器需求较少的业务，优点是降低了资源成本，部署方便；缺点是业务需要停止服务，如某些证券公司在不开盘的时候都会停服维护。

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

## CICD工具
### GitOps


测试资源：Infrastructrue as a Service ( IaaS )
生产资源：Infrastructure as Code ( IaC )，半自动化
### GitLab
### JenkinsX
### Tekton
### Drone
### Spinnaker
### Argo