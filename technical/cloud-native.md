# Cloud Native

__docker? swarm? compose? !!!__

 XEN???
 mereos???

 iaas
 paas

---
# k8s
+ 为什么 k8s 这么受欢迎
    * deployment
        - `kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80`
        - 一个命令增加10个nginx节点，线上运行，f5、A10 传统硬件负载均衡做不到
            + 应对流量洪峰，响应、调度
+ CNCF（Cloud Native Computing Foundation）最重要的组件之一, Cloud Native 应用的基石

---

# Cloud Native 的定义
+ 在动态环境中构建和运行可弹性扩展的应用
+ 代表技术包括容器、服务网格、微服务、不可变基础设施和声明式 API
+ 能够构建容错性好、易于管理和便于观察的松耦合系统
+ 结合可靠的自动化手段，可以使开发者轻松地对系统进行频繁并可预测的重大变更

---
# 是否应该迁移到云原生应用架构？

---
# 巴比特目前的架构
+ 镜像
+ 使用阿里云提供的 RDS、MySQL、负载均衡

# 存在的问题
+ 不利于升级管理
+ 依赖性
+ 不利于迁移
    * 之前的 青云 -> 阿里
        - 流量

---
# 回顾历史

为保证开发环境的正确（Bug 不是环境因素造成），想出一系列的隔离方式：虚拟机、容器虚拟化、语言虚拟机、应用容器（Java Tomcat）、虚拟环境（Python virtualenv），甚至是独立于语言的 DSL。

---

从最早的物理服务器开始，我们都在不断地抽象或者虚拟化服务器。
![虚拟化](https://chrislinn.ink/img/ops/server-growth.jpg)

---

+ 我们使用 XEN、KVM 等虚拟化技术，隔离了硬件以及运行在这之上的操作系统。
+ 我们使用云计算进一步地自动管理这些虚拟化的资源。
+ 我们使用 Docker 等容器技术，隔离了应用的操作系统与服务器的操作。
+ 有了 Serverless，我们可以隔离操作系统，乃至更底层的技术细节。

---
# 容器

隔离的开发测试环境和持续集成环境

+ 轻量级
+ 易于配置
+ 易于使用

---
# 容器生态
+ 镜像仓库
+ 服务编排
+ 安全管理
+ 持续集成与发布
+ 存储
+ 网络管理
+ ...

---
# 容器生态

https://chrislinn.ink/img/ops/container-ecosys.jpg


---

# docker 存在的问题
iptables 规则冲突，内部提供服务的端口被暴露在公共网络上

# 解决办法
https://chaifeng.com/to-fix-ufw-and-docker-security-flaw-without-disabling-iptables/

---
# Kubernetes (k8s)

+ 让容器应用进入大规模工业生产
+ 在单机上运行容器，无法发挥它的最大效能，只有形成集群，才能最大程度发挥容器的良好隔离、资源分配与编排管理的优势

---
# k8s
+ 容器编排调度引擎/系统
+ 一个规范
    * 描述集群的架构
    * 定义服务的最终状态
        - 将系统自动地达到和维持在这个状态
+ 云原生应用的基石
    * 相当于一个云操作系统

---
# 编排管理
Swarm、Mesos和Kubernetes

---
![k8s-arch](https://chrislinn.ink/img/ops/k8s-arch.jpg)

---

显示了组件之间交互的接口CNI、CRI、OCI等，这些将Kubernetes与某款具体产品解耦，给用户最大的定制程度，使得Kubernetes有机会成为跨云的真正的云原生应用的操作系统。

---
<!-- 
# 一些概念
+ deployment
    * 滚动升级和回滚应用
        - `kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1`
    * 扩容和缩容
        - `kubectl scale deployment nginx-deployment --replicas 10`
        - `kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80`

---

+ ingress?
+ pod
+ volume
+ secret
+ job

---
# 一些工具
+ kubectl
+ kubeadm
 -->

---
# cloud native 的核心目标
+ 云已经可以为我们提供稳定可以唾手可得的基础设施，但是业务上云成了一个难题
+ Kubernetes的出现与其说是从最初的容器编排解决方案，倒不如说是为了解决应用上云这个难题
    * 不是为了部署和管理容器而用Kubernetes，承载其上的应用才是 [价值所在]
    * 基于k8s“操作系统”之上构建的适用于不同场景的应用将成为发展方向

---
# 云原生应用的三大特征
+ 容器化包装：软件应用的进程应该包装在容器中独立运行。
+ 动态管理：通过集中式的编排调度系统来动态的管理和调度。
+ 微服务化：明确服务间的依赖，互相解耦。


---
# 微服务架构
+ 开发和部署上的灵活性和技术多样性
+ 带来了服务调用的开销、分布式系统管理、调试与服务治理方面的难题

---
# 微服务中最基础的服务注册发现
+ 客户端服务发现
    * Java应用中常用的方式是使用Eureka和Ribbon做服务注册发现和负载均衡
+ 服务端服务发现
    * Kubernetes中可以使用DNS、Service和Ingress来实现
        - 不需要修改应用代码，直接从网络层面来实现

---
![service-discovery-in-microservices](https://chrislinn.ink/img/ops/service-discovery-in-microservices.png)

---
# 云原生所需要的能力和特征

![cloud-native-architecutre-mindnode](https://chrislinn.ink/img/ops/cloud-native-architecutre-mindnode.jpg)

---
# 使用Kubernetes构建云原生架构
![create-k8s-native](https://chrislinn.ink/img/ops/create-k8s-native.jpg)

---

![build-cf-with-k8s](https://chrislinn.ink/img/ops/build-cf-with-k8s.jpg)


---
# service mesh
linkerd vs k8s vs linkerd2

---

# 如何迁移到云原生应用架构
https://jimmysong.io/migrating-to-cloud-native-application-architectures/


---

# 来得及吗，会不会被淘汰

---
# 云原生应用的架构
+ micro-service
+ serverless


---
# serverless

---

# 架构、数据库、缓存优化
+ https://tech.youzan.com/
+ https://tech.meituan.com/
+ http://mysql.taobao.org/monthly/
+ 阿里巴巴Java 开发手册
+ https://github.com/superhj1987/pragmatic-java-engineer