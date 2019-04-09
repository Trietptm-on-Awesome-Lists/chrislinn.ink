+ https://www.tarlogic.com/en/blog/how-kerberos-works/
+ https://github.com/serverless/serverless
+ https://www.phodal.com/blog/serverless-architecture-what-is-serverless-architecture/

serverless

+ http://dockone.io/article/1460
+ http://blog.ucloud.cn/archives/2476

+ https://jimmysong.io/kubernetes-handbook/cloud-native
+ http://www.servicemesher.com/awesome-servicemesh


istio

+ http://www.servicemesher.com/istio-handbook/
+ https://jimmysong.io/istio-handbook/
+ https://istio.io/zh/
+ https://github.com/servicemesher/istio-knowledge-map


istio vs linkerd

+ https://kubedex.com/istio-vs-linkerd-vs-linkerd2-vs-consul/
+ https://glasnostic.com/blog/comparing-service-meshes-linkerd-vs-istio
+ https://medium.com/devopslinks/istio-step-by-step-part-05-istio-and-linkerd-570b511f7a85
+ https://medium.com/@ihcsim/linkerd-2-0-and-istio-performance-benchmark-df290101c2bb
+ https://searchitoperations.techtarget.com/news/252457734/Linkerd-vs-Istio-fray-dominates-service-mesh-battle

sidecar?

Kubernetes Ingress vs Istio Gateway


k8s

+ http://docs.kubernetes.org.cn/
+ https://www.kubernetes.org.cn/doc-11
+ https://zhuanlan.zhihu.com/p/42507941
+ https://juejin.im/post/5c6657fdf265da2db3053dd9
+ https://blog.51cto.com/wzlinux/2321293
+ https://www.infoq.cn/article/get-along-well-with-kubernetes
+ https://tech.antfin.com/articles/224
+ https://www.cnblogs.com/along21/p/10297756.html





do this

+ docker & iptables
+ https://jimmysong.io/posts/what-is-a-service-mesh/
+ https://jimmysong.io/posts/from-kubernetes-to-cloud-native/
    * https://jimmysong.io/posts/what-is-serverless/
        - 应用必须运行在不论是实体还是虚拟的服务器上，必须经过部署、配置、初始化才可以运行，还需要对服务器和应用进行监控和管理，还需要保证数据的安全性，这些云能够帮我们简化吗？让我们只要关注自己代码的逻辑就好了，其它的东西让云帮我实现就好了。
        - Serverless架构是云的自然延伸
            + 云: IaaS -> PaaS,docker -> Serverless(BaaS/FaaS)
    * https://jimmysong.io/posts/microservice-reading-notes/
        - 如何架构
+ https://jimmysong.io/kubernetes-handbook/
+ https://jimmysong.io/kubernetes-handbook/appendix/kubernetes-and-cloud-native-summary-in-2017-and-outlook-for-2018.html
* https://jimmysong.io/posts/kubernetes-and-cloud-native-outlook-2019/
    - Kubernetes、Service Mesh、Istio 三者之间的关系：
        + Kubernetes 负责应用的生命周期管理，最小的治理单元是 Pod；
        + Service Mesh 解决服务间的流量治理，最小的治理单元是 Service（可以类比为 Kubernetes 中 Service 资源）；
        + 而 Serviceless 是更高一层的抽象，最小的治理单元是 APP；
    - 越向上层就越不关心应用的底层实现，到了 Serverless 开发者只需要关心代码逻辑，其他的一起都是配置
    - 2019年才开始学 Kubernetes 依然不晚，这可能是影响云计算未来10年的技术，甚至有人预测，未来的开发者可能一上手就是在云上开发，从提交代码、测试到发布一气呵成，直接基于 Git 操作即可完成，完全感受不到 Kubernetes 的存在。展望2019年，我在2017年的预测的趋势依然不变，2019年将更加深化。如果以 Kubernetes 的发展阶段类比，就像2017年时的 Kubernetes 一样，在一部分企业中 Service Mesh 已经开始快速落地，而 Knative 更像 2015 年时的 Kubernetes，一起才刚刚开始，毕竟也是 2018 年中才开源。
* https://jimmysong.io/posts/service-mesh-the-microservices-in-post-kubernetes-era/
* https://jimmysong.io/posts/istio-handbook-by-servicemesher/
* https://jimmysong.io/posts/istio-knowledge-map-and-handbook/
* https://www.cnblogs.com/163yun/p/8962278.html
* https://jimmysong.io/posts/istio-installation/
+ https://github.com/rootsongjc/
+ migrating-to-cloud-native-application-architectures
+ https://github.com/rootsongjc/cloud-native-sandbox
+ https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster/blob/master/README-cn.md
+ https://jimmysong.io/posts/envoy-proxy-config-deep-dive/
    * https://jimmysong.io/posts/envoy-sidecar-injection-in-istio-service-mesh-deep-dive/
* https://jimmysong.io/posts/istio-service-and-traffic-model/


---

## Cloud Native


## Serverless（无服务器）架构

第一次部署和第二次、第三次部署没有什么区别。

+ Baas
+ FaaS

---

Function-as-a-Service 按需计算

---

Serverless（无服务器架构）指的是由开发者实现的服务端逻辑运行在无状态的计算容器中，它由事件触发， 完全被第三方管理，其业务层面的状态则被开发者使用的数据库和存储资源所记录

下图来自谷歌云平台官网，是对云计算的一个很好的分层概括，其中 serverless 就是构建在虚拟机和容器之上的一层，与应用本身的关系更加密切。

![google-cloud](/img/ops/google-cloud.jpg)


---
## Pros
+ 降低成本
    * 版本管理服务器、持续集成服务器、测试服务器、应用版本管理仓库、数据库服务器等
    * 邮件服务、短信服务
    * 降低开发成本，只需要在配置文件上写下，这个数据库的表名，那么我们的数据就会存储到对应的数据库里
+ 快速上线
    * 内建自动化部署
    * 只需要关注于如何更好去实现业务
    * 使用测试来保证健壮性，那么结合持续集成，我们就可以在 PUSH 代码的时候，直接部署到生产环境
+ 系统安全性
    * 无服务器，无登陆入口
+ 适应微服务架构
+ 自动扩展能力

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
## K8S


---

>开发人员为了保证开发环境的正确（即，这个 Bug 不是环境因素造成的），想出了一系列的隔离方式：虚拟机、容器虚拟化、语言虚拟机、应用容器（如 Java 的 Tomcat）、虚拟环境（如 Python 中的 virtualenv），甚至是独立于语言的 DSL。


从最早的物理服务器开始，我们都在不断地抽象或者虚拟化服务器。

---

![虚拟化](/img/ops/server-growth.jpg)

---

+ 我们使用 XEN、KVM等虚拟化技术，隔离了硬件以及运行在这之上的操作系统。
+ 我们使用云计算进一步地自动管理这些虚拟化的资源。
+ 我们使用 Docker 等容器技术，隔离了应用的操作系统与服务器的操作。
+ 有了 Serverless，我们可以隔离操作系统，乃至更底层的技术细节。

