# Cloud Native

---
# k8s
+ 为什么 k8s 这么受欢迎
    * deployment
        - 一个命令增加10个nginx节点，线上运行，f5、A10 传统硬件负载均衡做不到
            + 应对流量洪峰，响应、调度
+ CNCF（Cloud Native Computing Foundation）最重要的组件之一, Cloud Native 应用的基石

---

# deployment 示例
* 滚动升级和回滚应用
    - `kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1`
* 扩容和缩容
    - `kubectl scale deployment nginx-deployment --replicas 10`
    - `kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80`

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

+ 使用 XEN、KVM 等虚拟化技术，隔离了硬件以及运行在这之上的操作系统。
+ 使用云计算进一步地自动管理这些虚拟化的资源。
+ 使用 Docker 等容器技术，隔离了应用的操作系统与服务器的操作。
+ 使用 Serverless 隔离操作系统，乃至更底层的技术细节。

---
# 目标
将软件以及它运行安装所需的一切文件（代码、运行时、系统工具、系统库）打包到一起，使得对于应用的构建、发布、运行更加敏捷和可控。


---
# 虚拟机
模拟一个完整的操作系统

+ 资源占用多
+ 冗余步骤多
+ 启动慢
+ 体积大

---
# 容器

Linux 容器, Linux Containers (LXC)

对进程进行隔离, 接触到的各种资源都是虚拟的

---
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
# 云计算
+ IaaS
    * 服务器租赁并提供基础设施外包服务
    * 亚马逊的EC2、S3存储
+ PaaS
    * 工具和服务(操作系统安装、监控和服务发现)的集合, 用户只需要部署自己的应用即可
        - Google App Engine
+ SaaS
    * 终端用户可以直接使用的应用程序

---

# docker
容器的一种封装，提供简单易用的容器使用接口。目前最流行的容器解决方案。

+ 基于 LXC
+ 利用 namespaces 来做权限的控制和隔离
+ 利用 cgroups 来进行资源的配置
+ 通过 aufs 提高文件系统的资源利用率

---
主要用途:

+ 提供一次性的环境。如本地测试他人的软件、持续集成的时候提供单元测试和构建的环境
+ 提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容
+ 组建微服务架构。通过多个容器，一台机器可以跑多个服务，在本机就可以模拟出微服务架构

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

+ CRI（Container Runtime Interface）：容器运行时接口，提供计算资源
+ CNI（Container Network Interface）：容器网络接口，提供网络资源
+ CSI（Container Storage Interface）：容器存储接口，提供存储资源

---
# 核心组件
+ etcd 用于保存集群所有的网络配置和对象的状态信息
+ apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
+ controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
+ scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
+ kubelet负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；
+ Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）；
+ kube-proxy负责为Service提供cluster内部的服务发现和负载均衡；

---
# Kubernetes中的资源隔离层次
保证对集群资源的最大化和最优利用率

+ 容器
+ Pod：命名空间的隔离(限制用户空间和资源限额)，资源隔离和调度的最小单位，共享网络，每个Pod都有独立IP，使用Service Account为Pod赋予账户
+ Sandbox：是对最小资源调度单位的抽象，甚至可以是虚拟机
+ Node：网络隔离，每个节点间网络是隔离的，每个节点都有单独的IP地址
+ Cluster：元数据的隔离，使用Federation可以将不同的集群联合在一起

---
# cloud native 的核心目标
+ 云已经可以为我们提供稳定可以唾手可得的基础设施，但是业务上云成了一个难题
+ Kubernetes的出现与其说是从最初的容器编排解决方案，倒不如说是为了解决应用上云这个难题
    * 不是为了部署和管理容器而用Kubernetes，承载其上的应用才是 [价值所在]
    * 基于k8s“操作系统”之上构建的适用于不同场景的应用将成为发展方向

---
# 云原生应用应该具备
+ 敏捷
+ 可靠
+ 高弹性
+ 易扩展
+ 故障隔离保护
+ 不中断业务持续更新

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
![microservice-concern](https://chrislinn.ink/img/cloud-native/microservice-concern.jpg)

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
在现今的复杂的大型网站情况下，单体应用被分解为众多的微服务，服务之间的依赖和通讯十分复杂。

可以将它比作是应用程序或者说微服务间的 TCP/IP，负责服务之间的网络调用、限流、熔断和监控。

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
+ 解耦应用程序的重试/超时、监控、追踪和服务发现
+ 扩展kubernetes的应用负载均衡机制，实现灰度发布
+ 完全解耦于应用，应用可以无感知，加速应用的微服务和云原生转型

---

Kubernetes中的应用将作为微服务运行，但是Kuberentes本身并没有给出微服务治理的解决方案，比如服务的限流、熔断、良好的灰度发布支持等。

kube-proxy里基于iptables的原生负载均衡，并且服务间的流量也没有任何控制。

---
# 早期的 service mesh
twitter 开发的 Finagle、Netflix 开发的 Hystrix 和 Google 的 Stubby ，适用于特定的环境和特定的开发语言，并不能作为平台级的 service mesh 支持。

# Linkerd vs Istio vs Linkerd2

---
使用Vistio监控Istio服务网格中应用程序和集群之间的网络流量


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
# 高可用
Keepalived + Haproxy

---
# 持续集成与发布
![build-cf-with-k8s](https://chrislinn.ink/img/cloud-native/build-cf-with-k8s.jpg)

---
# 日志收集与监控
![elk](https://chrislinn.ink/img/cloud-native/filebeat-log-collector-arch.png)

---
# 集群监控
![heapster](https://chrislinn.ink/img/cloud-native/kubernetes-heapster-monitoring.png)

---
# 应用监控
![app-mon](https://chrislinn.ink/img/cloud-native/kubernetes-app-monitoring.png)

---
# 分布式追踪
将单体应用拆成多个微服务之后，监控服务之间的依赖关系和调用链，以判断应用在哪个服务环节出了问题，哪些地方可以优化

OpenTracing 是 CNCF 提出的分布式追踪的标准

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
# k8s & Cloud Native 上手试玩
+ `kubectl` CLI
+ 生产级的 Kubernetes 简化管理工具 `kubeadm`, 快速部署
+ https://jimmysong.io/kubernetes-handbook/cloud-native/play-with-kubernetes.html
+ https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster
+ https://github.com/rootsongjc/cloud-native-sandbox

---
# 云原生应用的架构
+ micro-service
    * 与 cloud native 自然贴合
+ serverless
    * 也算是 微服务的一种

---
# micro-service
## pros
+ 关注于一个业务功能，松耦合，灵活性更高
+ 每个服务按需定制扩展
+ 可由不同团队独立开发，互不影响
+ 独立测试、部署、升级、发布, 持续交付,允许在频繁发布不同服务的同时保持系统其他部分的可用性和稳定性
+ 提高容错性, 不会让整个系统瘫痪
+ 不被技术栈限制

---
# micro-service
## cons
+ 可能带来代码重复
+ 提高了系统的复杂度
+ 引入分布式系统的复杂性
+ 往往使用异步编程、消息与并行机制，如果应用存在跨微服务的事务性处理，其实现机制会变得复杂化
+ 服务的注册与发现问题
+ 运维开销及成本增加

---
# serverless
Serverless（无服务器架构）指的是由开发者实现的服务端逻辑运行在无状态的计算容器中，它由事件触发， 完全被第三方管理，其业务层面的状态则被开发者使用的数据库和存储资源所记录

---
Serverless 是构建在虚拟机和容器之上的一层，与应用本身的关系更加密切。

![google-cloud](https://chrislinn.ink/img/cloud-native/google-cloud.jpg)


---
# 分类
* BaaS（Backend as a Service）
    - 一个个的API调用后端或别人已经实现好的程序逻辑
    - 身份验证服务Auth0
    - RDS
* FaaS（Functions as a Service)
    - 传统的服务器端软件一般需要长时间驻留在操作系统的虚拟机或者容器中运行
    - FaaS是直接将程序部署上到平台上，当有事件到来时触发执行，执行完了就可以卸载掉。
        + 事件驱动的由消息触发的服务
        + 按需计算

---
## Pros
+ 降低成本
    * 版本管理服务器、持续集成服务器、测试服务器、应用版本管理仓库、数据库服务器等
    * 邮件服务、短信服务
    * 降低开发成本，只需在配置文件上写下数据库的表名，数据就会存储到对应的数据库里
+ 快速上线
    * 内建自动化部署
    * 只需要关注于如何更好去实现业务
    * 使用测试来保证健壮性，结合持续集成，可以在 PUSH 代码时直接部署到生产环境
+ 系统安全性
    * 无服务器，无登陆入口
+ 适应微服务架构
+ 自动扩展能力

---
## Cons
+ 不适合长时间运行应用
    * 应用不运行的时候进入 “休眠状态”
        - 冷启动时间
    * 长期租车的成本肯定比买车贵
+ 依赖第三方服务
    * 不利于迁移
    * 建立隔离层: API 网关, 数据库层...
        - 带来的问题会比解决的问题多
+ 缺乏调试和开发工具
    * 每次调试时需要一遍又一遍地上传代码
    * 分级别纪录日志: error、warn、info、verbose、debug、silly

---

# 架构、数据库、缓存优化
+ https://tech.youzan.com/
+ https://tech.meituan.com/
+ http://mysql.taobao.org/monthly/
+ 阿里巴巴Java 开发手册
+ https://github.com/superhj1987/pragmatic-java-engineer