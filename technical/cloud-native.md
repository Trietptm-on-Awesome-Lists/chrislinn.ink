# Cloud Native

---
# k8s

__docker? swarm? compose? !!!__

+ 为什么 k8s 这么受欢迎
    * deployment
        - ``
        - `kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80`
    * swarm, mereos
+ CNCF（Cloud Native Computing Foundation）最重要的组件之一, Cloud Native 应用的基石

---

# Cloud Native 的定义
+ 在动态环境中构建和运行可弹性扩展的应用
+ 代表技术包括容器、服务网格、微服务、不可变基础设施和声明式 API

---
迁移到云原生应用架构？

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
从最早的物理服务器开始，我们都在不断地抽象或者虚拟化服务器。

![虚拟化](https://chrislinn.ink/img/ops/server-growth.jpg)

---

+ 我们使用 XEN、KVM等虚拟化技术，隔离了硬件以及运行在这之上的操作系统。
+ 我们使用云计算进一步地自动管理这些虚拟化的资源。
+ 我们使用 Docker 等容器技术，隔离了应用的操作系统与服务器的操作。
+ 有了 Serverless，我们可以隔离操作系统，乃至更底层的技术细节。

---
# docker

---

# docker 存在的问题
iptables?

---
## Kubernetes (k8s)
+ 容器编排调度引擎/系统
+ 一个规范
    * 描述集群的架构
    * 定义服务的最终状态
        - 将系统自动地达到和维持在这个状态
+ 云原生应用的基石
    * 相当于一个云操作系统

---

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

---
# service mesh
linkerd vs k8s vs linkerd2

---

# 如何迁移到云原生应用架构
https://jimmysong.io/migrating-to-cloud-native-application-architectures/

---
# serverless


---

# 来得及吗，会不会被淘汰


---

# 架构、数据库、缓存优化
+ https://tech.youzan.com/
+ https://tech.meituan.com/
+ http://mysql.taobao.org/monthly/
+ 阿里巴巴Java 开发手册
+ https://github.com/superhj1987/pragmatic-java-engineer