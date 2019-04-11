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
![虚拟化](https://chrislinn.ink/img/cloud-native/server-growth.jpg)

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

https://chrislinn.ink/img/cloud-native/container-ecosys.jpg


---

# docker iptables 规则冲突问题
内部提供服务的端口被暴露在公共网络上

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
![k8s-arch](https://chrislinn.ink/img/cloud-native/k8s-arch.jpg)

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
# 云原生的设计理念

---

+ 面向分布式设计（Distribution）：容器、微服务、API 驱动的开发；
+ 面向配置设计（Configuration）：一个镜像，多个环境配置；
+ 面向韧性设计（Resistancy）：故障容忍和自愈；
+ 面向弹性设计（Elasticity）：弹性扩展和对环境变化（负载）做出响应；
+ 面向交付设计（Delivery）：自动拉起，缩短交付时间；
+ 面向性能设计（Performance）：响应式，并发和资源高效利用；
+ 面向自动化设计（Automation）：自动化的 DevOps；
+ 面向诊断性设计（Diagnosability）：集群级别的日志、metric 和追踪；
+ 面向安全性设计（Security）：安全端点、API Gateway、端到端加密；

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
![service-discovery-in-microservices](https://chrislinn.ink/img/cloud-native/service-discovery-in-microservices.png)

---
# 云原生所需要的能力和特征

![cloud-native-architecutre-mindnode](https://chrislinn.ink/img/cloud-native/cloud-native-architecutre-mindnode.jpg)

---
云端架构(分布式系统)，相对单体架构来说会带来很多挑战。一致性、延迟和网络分区、服务监控的变革、服务暴露、权限的管控等。

---

+ 版本化和分布式配置
+ 服务注册发现
+ 路由和负载均衡
+ 容错
    * 熔断器: 阻绝该服务与其依赖的远程调用
    * 隔板: 将服务分区，以便限制错误影响的区域
+ API 网关/边缘服务
    * 针对其开发微服务，尝试解决延迟、往返通信开销(减少服务端调用)、设备&协议多样性

---

# Service Mesh
Kubernetes中的应用将作为微服务运行，但是Kuberentes本身并没有给出微服务治理的解决方案，比如服务的限流、熔断、良好的灰度发布支持等。

---

# Service Mesh 可以用来做什么
+ Traffic Management：API网关
+ Observability：服务调用和性能分析
+ Policy Enforcement：控制服务访问策略
+ Service Identity and Security：安全保护

---
# Service Mesh 的特点
+ 专用的基础设施层
+ 轻量级高性能网络代理
+ 提供安全的、快速的、可靠地服务间通讯
+ 扩展kubernetes的应用负载均衡机制，实现灰度发布
+ 完全解耦于应用，应用可以无感知，加速应用的微服务和云原生转型

---

# Linkerd vs Istio vs Linkerd2

---
# kubernetes 应用开发部署流程

https://jimmysong.io/kubernetes-handbook/guide/deploy-applications-in-kubernetes.html

---
![how-to-use-kubernetes-with-istio](https://chrislinn.ink/img/cloud-native/how-to-use-kubernetes-with-istio.jpg)

---
# kubernetes 应用开发部署流程
+ 开发容器化应用
+ 使用Wercker持续集成工具构建docker镜像上传到docker镜像仓库中
+ 在本地使用docker-compose测试
+ 使用kompose自动生成kubernetes的yaml文件
+ 注入Envoy sidecar容器，集成Istio service mesh中


---
# 架构选型

---

![pick1](https://chrislinn.ink/img/cloud-native/pick1.jpg)

---
# 方案调研
https://chrislinn.ink/img/cloud-native/selection.jpg


---
# 使用Kubernetes构建云原生架构
![create-k8s-native](https://chrislinn.ink/img/cloud-native/create-k8s-native.jpg)

---

![build-cf-with-k8s](https://chrislinn.ink/img/cloud-native/build-cf-with-k8s.jpg)

---
# 如何迁移到云原生应用架构

![migrating-monolith-to-kubernetes](https://chrislinn.ink/img/cloud-native/migrating-monolith-to-kubernetes.jpg)

---
# 迁移步骤
+ 将原有应用拆解为服务
+ 制作 docker 镜像
+ 制作应用的配置文件
+ 制作 Kubernetes YAML 文件
+ 制作 Bootstrap 脚本
    * 在启动时根据 Pod 的环境变量、主机名或其他可以区分不同 Pod 和将启动角色的变量来修改配置文件和启动服务应用
+ 制作 ConfigMaps
    * 将 Hadoop 的配置文件和 bootstrap 脚本作为 ConfigMap 资源保存，用作 Pod 启动时挂载的 volume

---
# spark-on-yarn 例子
![spark-on-yarn-with-kubernetes](https://chrislinn.ink/img/cloud-native/spark-on-yarn-with-kubernetes.png)

---

# 来得及吗，会不会被淘汰

> 2019年才开始学 Kubernetes 依然不晚，这可能是影响云计算未来10年的技术，甚至有人预测，未来的开发者可能一上手就是在云上开发，从提交代码、测试到发布一气呵成，直接基于 Git 操作即可完成，完全感受不到 Kubernetes 的存在。展望2019年，我在2017年的预测的趋势依然不变，2019年将更加深化。如果以 Kubernetes 的发展阶段类比，就像2017年时的 Kubernetes 一样，在一部分企业中 Service Mesh 已经开始快速落地，而 Knative 更像 2015 年时的 Kubernetes，一起才刚刚开始，毕竟也是 2018 年中才开源。
> 
> --蚂蚁金服 宋净超

---
+ 之前还不成熟，现在生态正好稳定
+ 没有淘不淘汰，只有合不合适
+ knative 原生调度最节省资源 (使用YARN调度 vs standalone调度 vs kubernetes原生调度)


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