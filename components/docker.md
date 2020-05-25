<!--
 * @Author: howardhh
 * @Date: 2020-05-22 08:42:09
 * @LastEditTime: 2020-05-25 13:46:51
 * @LastEditors: Please set LastEditors
 * @Description: Docker Learning
 * @FilePath: \kubernetes-tutorials\components\docker.md
--> 

# Docker
<hr>

## 概述
Docker 是一个基于 Go 语言开发的开源容器引擎，可以让你将应用与基础设施分开，从而快速的部署到其他环境。

### 容器
Docker 是容器的一种更高级的实现方式，也可以说是容器的代名词，它的前身是 LXC（Linux Container），诞生于2008年。
LXC 是一种内核轻量级的操作系统层虚拟化技术，可以提供比传统虚拟化更轻量级的虚拟化，主要基于 Namespace 和 Cgroup 两大机制。

容器的本质是<b>基于镜像的跨环境迁移</b>。
DevOps、微服务与容器是相辅相成，不可分割的。
不是微服务架构，基本不需要容器，虚拟机就能搞定，不需要 DevOps，一年部署一次，开发和运维沟通再慢都能搞定。所以，容器的本质是基于镜像的跨环境迁移。镜像是容器的根本性发明，是封装和运行的标准。

### Docker 引擎
Docker 引擎是一个 C/S 架构的应用，包含如下主要组件：

* 服务端，运行在操作系统中的守护进程（<code>dockerd</code>）
* REST API，提供客户端操作服务端的接口
* 客户端，命令行工具（<code>docker</code>）

![engine-components](../pictures/engine-components-flow.png)

### Docker 架构
* The Docker daemon
* The Docker client
* Docker objects
  * IMAGES：镜像是用来创建容器的只读模板，通常是基于另外一个镜像，并添加一些自定义的内容。比如，一个镜像可以包含一个完整的 centos 操作系统，里面安装 apache 和用户的应用，以及一些配置参数来使容器运行。用户既可以自己创建镜像，也可以直接从其他人那里下载一个已经做好的镜像来直接使用。[DockerHub](https://hub.docker.com/)
  * CONTAINERS：容器是从镜像创建的运行实例，它可以被启动，开始、停止、删除、每个容器都是互相隔离的，保证安全的平台，可以把容器看做是要给简易版的linux环境（包括root用户权限、镜像空间、用户空间和网络空间等）和运行再其中的应用程序。
* Docker registry：镜像仓库是集中存储镜像的地方，分为公共仓库和私有仓库。
![docker-architecture](../pictures/docker-architecture.svg)


### Docker 优势
* 快速、持续的交付应用
* 响应式的部署和扩展
* 在相同硬件上运行更多负载

<b>Build, Ship, Run</b>

[zz举个例子](https://www.zhihu.com/question/28300645/answer/67707287)：
我来到一片空地，想建个房子，于是我搬石头、砍木头、画图纸，一顿操作，终于把这个房子盖好了。

![zhihu1](../pictures/docker-zhihu1.jpg)

结果，我住了一段时间，想搬到另一片空地去。这时候，按以往的办法，我只能再次搬石头、砍木头、画图纸、盖房子。
但是，跑来一个老巫婆，教会我一种魔法。
这种魔法，可以把我盖好的房子复制一份，做成“镜像”，放在我的背包里。

![zhihu2](../pictures/docker-zhihu2.jpg)

我到了另一片空地，就用这个“镜像”，复制一套房子，摆在那边，拎包入住。

![zhihu3](../pictures/docker-zhihu3.jpg)

<hr>

## 实战

### 安装

* [Mac](https://docs.docker.com/docker-for-mac/install/) or [Windows](https://docs.docker.com/docker-for-windows/install/)：下载 Docker Desktop
* [Linux](https://docs.docker.com/engine/install/binaries/)：通过不同发行版系统的软件源（<code>yum</code> / <code>apt-get</code>）或二进制包安装。证通开发测试环境[点此](http://11.8.38.55:9080)安装。

### 镜像操作
```bash
docker pull 
docker push
docker image ls
```

### 容器操作
```bash
docker run -d --name tomcat-demo -p 8080:8080 tomcat:8
```

### 构建应用镜像
Docker 使用<code>Dockerfile</code>构建镜像。比如我们基于 [tomcat](https://hub.docker.com/_/tomcat?tab=description) 镜像构建一个 java 应用：
```
FROM tomcat:8
```

### 容器编排
目前容器编排比较常用的有如下三个工具：
* Docker-Compose：Docker 提供的一个命令行工具，用来定义和运行多个容器组成的应用程序。仅能管理当前主机上的 Docker。
* Docker Swarm：Docker 公司自研的用来管理多主机上 Docker 容器的工具。
* Kubernetes（K8S）：定位与 Docker Swarm 类似，已成为容器编排领域的领导者。

### DevOps