# Bytom

## 我在比原的工作

+ bytom
    * 可以看看 release notes
    * PoW 相关完善及优化
        + 优化tensority
            * go 920ms -> go 820ms -> simd 160ms -> cuda 6ms
        + get work 定时更新变为出块即更新，更加及时
    - [x] consume newBlockCh for vault_mode
        + 被马总解决了
    - block_recommit 交易数据及时入块
        + 其实就是用 map
        + ticker 定期生成 NewBlockTemplate，因为 新添交易的话 commitment 会改变，blockheader 也会改变，不用 map 的话，submit work不对应
        + 老 map 要注意 GC
    - 全局交易索引
        + config
        + test
+ Precogs
    * 涉及 P2P
+ 中心化钱包
    * api & database schema
    * build tx
        - utxo >21 build fail
        - retire类型交易 utxo 是否忘了处理。应该变成不可用。
    * fee estimate
    * multisign
        - [ ] delete unconfirmed utxo
        - [ ] 调研if txProposalSign.Signatures == "" {是否可以去除
    * all utxos
        - check `utxo.Asset.Asset` in `btm.go`
        - raw sql
        - address str for address
    * 关于null的索引不生效的传说是真的么?
        - 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描
        - https://www.v2ex.com/t/60437
        - https://dev.mysql.com/doc/refman/5.7/en/is-null-optimization.html
    * TEXT
        - varchar 是可变长字符串,不预先分配存储空间,长度不要超过 5000,如果存储长度大于此值,定义字段类型为 text ,独立出来一张表,用主键来对应,避免影响其它字段索引效率。
        - 定长，累积很占空间
        - utxo use hash for cp? 
        - server-side max exec time
        - https://dev.mysql.com/doc/refman/8.0/en/full-text-adding-collation.html
    * redis

<!-- 

## 下版本
+ 应该区分用户提交的业务形态的交易（并在数据库中存一份）和提交的 raw tx。
++ 这么做也有利于展示交易被回滚的情况，以告知用户。不然用户发了一笔交易，发生回滚就突然不见了。
+ utxo 现在是单纯的 10 min lock_until，而 unconfirmed tx 现在又不更新 utxo，应该设置长一点防止 出块超过 10min（现在这种情况在主网上还是很可能出现的）。这样可能导致间隔 10min 的两笔交易用了同样的 utxo，都能submit成功，但最终矿工只打包一个，用户莫名丢失了一笔交易但我们却没能提示。
++ 这块要好好再设计，不能等入块了才解锁，不然如果一直不上链就永远锁住了。应该做成 submit 给 bytomd 就锁住，入块了就设置is_spend，如果一直没入块后面 expired 了才解锁utxo 。做好 utxo 和 balance 的删除/更新。
++ 最好还是加上判断确认数（5～6次）才能进行花费 。
+ 应该区分 未确认交易 导致的 available balance 和 total balance
+ asset 价格 redis  查不到 应该重新拿，而不是查 mysql。asset 价格不放数据库。
+ sql IN 语句是否存在性能问题，是否用 数字型 而非字符串 加快查询
+ api 不应该和 bytomd 交互，应该做一个 channel/callback/mq 给 updater 或者 load balancer，统一和 bytomd 进行交互（即 所有和bytomd 的交互应该是一个统一的出入口）
++ bytomd 目前是单节点 没有 load balancing，应该做上以免 submit tx 造成的 ddos 或别的原因造成的不可用
+++ bytomd load balancing 要注意节点状态不一致的处理，从哪个节点同步数据。拿块可以通过判断最高高度，问题主要是 ws 去哪个节点拿 tx -->

<!-- 
## mining pool

+ MinDiff
+ DiffType
+ GetTargetHex
+ leading zero
+ retarget
+ processShare
    + s.hashrateExpiration, 
    + s.minuteHashrateExpiration
+ shareTimeRing

## Tensority  
### research
+ mulMatrix
    * __blas__
        - openblas
            + parameters tweaking
        - single type blas?
    * simd
        - __golang simd__
            + https://github.com/bjwbell/gensimd
            + https://github.com/mengzhuo/intrinsic
            + https://github.com/yesuu/simd/blob/master/mul_amd64.s
            + https://github.com/rkusa/gm
            + https://github.com/Everlag/goSIMD
            + https://www.google.com/search?q=go+simd&oq=go+simd&aqs=chrome..69i57j0l5.2290j0j7&sourceid=chrome&ie=UTF-8
            + https://www.google.com/search?q=yeppp&oq=yeppp&aqs=chrome..69i57j69i61j0l4.289j0j4&sourceid=chrome&ie=UTF-8
            + https://yushuangqi.com/blog/2016/go-ru-he-shi-yong--simd-zhi-ling.html
                * https://github.com/golang/go/blob/master/src/cmd/internal/obj/x86/asm6.go
            + https://www.cryptologie.net/article/406/simd-instructions-in-go/
            + https://golanglibs.com/top?q=simd
            + https://godoc.org/github.com/slimsag/rand/simd
            + https://github.com/sbinet/vector
            + https://github.com/reiver/go-float64x4
            + https://github.com/pennello/go_swar
            + should able to use int, save time for float64 conversion
        - cpp simd
        - gpu simd
    * opencv
    * cuda
        - https://archive.fosdem.org/2014/schedule/event/hpc_devroom_go/attachments/slides/486/export/events/attachments/hpc_devroom_go/slides/486/FOSDEM14_HPC_devroom_14_GoCUDA.pdf
    * openmp
        - give up
            + weird `go build bytomd`
                * no need to provide `CGO_LDFLAGS="-g -O2 -fopenmp"`
            + goood things
                * go test&build need to provide `CGO_LDFLAGS="-g -O2 -fopenmp"`
    * eigen
    * opencl
+ mat_init
    * simd?
    * gpu simd?
+ 有没有 生成 doc 的文件
+ mining/tensority
+ 搞清楚 BigEdian LittleEdian 的区别

### DONE
+ Time & space opt for dataIdentity[] init in mulMatrix()
+ clean up code & update tensority test
+ confirm all use SHA-3-256 
+ no need for parallelize extSeed, as SHA-3-256 is fast enough
    * .
        ```
        // 67 ms
        cache := calcSeedCache(seed.Bytes())

        // 1.030978394s
        data := mulMatrix(hash.Bytes(), cache)

        // 191.217µs
        hashMatrix(data)

        dataIdentity time:  57.954µs
        result time:  501.596µs
        ui32 time:  3.719813ms
        f64 time:  71.828938ms
        loop tmp time:  845.702µs 
        loop sha3 time:  6.524µs
        1 loop ma time:  240.240435ms
        4 loops ma time:  918.162585ms
        wg 4 loops ma time:  821.528586ms
        ```
+ go vs cpp, single_thread vs multi_thread
    * 矩阵点乘因为 库的问题，cpp 的 openblas 不如  go 的gonum/mat 快 改成多线程并发以后也是  cpp 不如 go 好   cpp多线程反而比cpp单线程更慢了
    * 我的笔记本上
        - go 单线程 gonum\mat: 920ms
        - go 4线程 gonum\mat: 820ms
        - cpp 单线程 openblas: 2.4s
        - cpp 4线程 openblas: 5.3s (理论上cpp 4线程能优化到700ms，我也不知道我为什么写出来这么渣...)
+ cpp -O3
    * ` g++ byte_order.c sha3.c  test_BytomPoW.cpp -I /opt/OpenBLAS/include/ -L/opt/OpenBLAS/lib -lopenblas -lpthread -std=c++11 -pthread -mavx2 -O2`
        - mulMatrix: 2.53 -> 1.65
        - total: 2.98898s -> 2.05s
+ Kui's first cpp slower
    * 12.9 s
+ extend seed in cpp
+ cpp multi-thread slower
    * 2.4s vs 5.3s
+ Kui's second cpp
    * 160ms!
+ 现在做的东西也不知道有没有价值，还是在优化代码,主要是 区块验证/挖矿这块
还没有做 P2P, 也没有接触虚拟机
    * 北京那边发挥很不稳定
    * 2.8s 被优化到 12.9s
    * then 160ms!
        - GPU?
        - 多线程
        - SIMD
+ cpu flag
    * `cat /proc/cpuinfo`
+ toIdentityMatrix
    * 0.189s -> 0.172s
    * 17ms faster
+ SIMD
    * Beijing
        - mul 230ms
        - total 450ms
        - single-thread 365ms
    * combine
        - sThread 167ms
        - sThread total 289ms
        - sThread opt-init16 total 256ms
        - mThread 0.333546s
        - mThread total 0.429112s
+ cgo
    * mul 178ms
    * total 280ms
+ shared lib
    * dl
        - https://www.google.com/search?q=golang+c+shared+lib&oq=golang++c+shared+lib&aqs=chrome..69i57j69i60j0.4787j0j7&sourceid=chrome&ie=UTF-8
        - https://github.com/rainycape/dl
    * plugin
        - https://golang.org/pkg/plugin/
        - https://medium.com/learning-the-go-programming-language/writing-modular-go-programs-with-plugins-ec46381ee1a9
    * https://github.com/golang/go/issues/16805
    * https://www.ardanlabs.com/blog/2013/08/using-c-dynamic-libraries-in-go-programs.html
+ monero
    * Monero verification time
        - https://bitcointalk.org/index.php?topic=583449.0
        - https://www.reddit.com/r/Monero/
        - https://monero.stackexchange.com/
        - https://forum.getmonero.org/
        - https://mattermost.getmonero.org/login
        - https://telegram.me/bitmonero
+ bytes
    * It is analogous to the facilities of the strings package.
    * `func (b *Buffer) Bytes() []byte`
        - Bytes returns a slice of length b.Len() holding the unread portion of the buffer. The slice is valid for use only until the next buffer modification (that is, only until the next call to a method like Read, Write, Reset, or Truncate). The slice aliases the buffer content at least until the next buffer modification, so immediate changes to the slice will affect the result of future reads.
    * [为什么电脑数据一个字节是8位？](https://www.guokr.com/question/542532/)
+ tensor
    * [X]win64 ver
        - define flag
        - no need fPIC
    * [X]win32 ver
+ openmp
    * 4-core on 4-core
        - faster
    * 4-core on 1-core
        - no change
    * 1-core on 1-core
        - no change
    * 1-core on 4-core
        - faster
+ coinbase data
    * Getblocktemplate allow you to define coinbase. You can check btcpool code. In stratum.cc, we define coinbase. See initfromGbt function. gbt stands for getblocktemplate.
    * 看下btc的交易结构及coinbase交易. 没有pre tx所以 就可以利用这个字段来写自定义信息. 比特币是在coinbase交易的输入的脚本里写的.
 -->

<!-- 
# Why I don't like Bytom
If you look into the Bytom mining code, you will find it hard to understand. In fact, it's designed to collaborate with bitmain's hardware. How can a blockchain product be promising if it doesn't have its own right to choose the algo?
 --> 

## Bytom 架构

### 商业模型
> https://gguoss.github.io/2017/06/28/Bytom-s-data-structure/
> 
> __资产账户__
> ![bytom_asset](/img/bytom/bytom_asset.png)
> 
> + assetid 是全局唯一的资产识别id
> + alias 是资产的别名，可便于记忆，如(gold, silver) 
> + vmversion 是为了软分叉时，做到动态过度。
> + program 是指发布该资产时需要执行的程序
> + initialblockhash 是指该资产是在哪个块高度被登记
> + signer 管理公私钥对，以便用该资产的私钥签名，只有拥有该资产私钥的人才能发布该资产
> + definition 对该资产的解释说明等。

### 软件架构

#### 技术特点
* 区块处理
    - btc
* tx
    - butxo 支持多资产
        + "扩展 utxo，扩展字段不上链，相当于拆表"?"分两类，一类 上链的，一类钱包的"?
    - txin txout 分多 action、做映射
* merkle tree
    - trie (pat)
    - 2 叉，不平衡
    - 保存交易状态
* p2p
    - tendermint reactor模型
    - eth discover
    - eth block sync
* 合约
    - chain

#### BVM
+ 智能合约
    * 合约交易
        - Bytom 默认不支持含 btm 的交易对，只支持 非btm交易对
        - btm 只支持标准(P2SH)交易
        - 但有个 解决方案：将合约交易转为 P2SH 交易



## blockcenter

最近在负责　blockcenter　session　登录交互这一部分，记录一下开发时的所习所想。

主要基础知识还是来自：

+ 图解密码学
+ [Crypto 101](https://github.com/crypto101/book)
+ [Practical-Cryptography-for-Developers-Book](https://github.com/nakov/Practical-Cryptography-for-Developers-Book)
+ Applied Cryptography: Protocols, Algorithms, and Source Code in C 应用密码学
+ [An Intensive Introduction to Cryptography](https://intensecrypto.org/public/index.html)
+ [HTTP 身份验证](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication)

### Cross-Origin Resource Sharing (CORS)

XMLHttpRequest和Fetch API遵循同源策略。 这意味着使用这些API的Web应用程序只能从加载应用程序的同一个域请求HTTP资源，除非响应报文包含了正确CORS响应头。

跨域资源共享（ CORS ）机制 __允许__ Web 应用服务器进行跨域访问控制，从而使跨域数据传输 __得以安全__ 进行。现代浏览器支持在 API 容器中（例如 XMLHttpRequest 或 Fetch ）使用 CORS，以降低跨域 HTTP 请求所带来的风险。

#### 什么情况下需要 CORS

+ XMLHttpRequest 或 Fetch 发起的跨域 HTTP 请求
+ Web 字体 (CSS 中通过 @font-face 使用跨域字体资源), 因此，网站就可以发布 TrueType 字体资源，并只允许已授权网站进行跨站调用。
+ WebGL 贴图
+ 使用 drawImage 将 Images/video 画面绘制到 canvas
+ 样式表（使用 CSSOM）


## patent


### 利用网络弹性能源基础设施和物联网实现 AI 赋能的区块链智能合约

#### 技术领域
本说明书的一个或多个实例涉及区块链和人工智能计算技术领域，尤其涉及运行在区块链上的、通过人工智能硬件进行计算的智能合约，对能源和物联网进行自动化和现代化的方法。

#### 技术背景
区块链被定义为一个分布式数据库或数字账本，它使用固有的抗修改的加密签名记录价值交易。它维护一个不断增长的记录列表，称为块，每个块包含一个时间戳和到上一个块的链接，能够防止篡改和修订。

基于区块链的智能合约（在没有中介机构充当货币和信息仲裁者的情况下交换价值的技术或应用程序）可以在没有人工交互的情况下执行，数据更难修改，因为块中的数据不能进行追溯性修改。区块链赋能的智能合约展现了在不需要第三方信任机构的情况下价值交换的潜力，提高了安全性，优化交换和跟踪价值的能力。

人工智能可以通过学习数据，产生知识，并利用这些知识通过灵活适应实现特定目标和任务，自动做出优化的判断和决策。避免人类思考的局限，生成效率不够高的策略，或者灵活地根据场景进行决策，避免固定的逻辑。并可以自动化流程，将人类从反复决策的过程解放出来，实现 24 小时的可靠运行。


#### 发明内容
传统的物联网、电网中，系统的运行是中心化控制的、逻辑固定的。这个发明通过一个创新的区块链智能合约应用到电力基础设施，以及一系列网络化来实现生产、传输、分配和消耗能源以及价值的连接体。通过人工智能区块链解决方案加密、监控和自动化交易以及解除对第三方信任的依赖来提高网络弹性和优化分布式能源资源的复杂交换。通过物联网设备感知和交换信息，支持人工智能的区块链解决方案还可以帮助更好地分析来自数千个变量（工业控制系统异常、频率、负载和电压变化）的数据集，并将它们组织成加权关系，可通过区块链追溯这些加权关系。由于这些变量中的数据模式可以通过机器学习计算机神经网络更好地检测，可以更新区块链智能合约以更好地保护和交换能源数据。

具体来说，这个发明的创新设计是：

1. 传统系统使用中心化数据库，存在单点故障和信任忧患。而目前电网缺乏必要的安全性和恢复能力，无法防止对灾害应急响应系统、电网边缘设备和相关电力基础设施的网络攻击。这个专利通过使用区块链的数字账本和加密签名交易数据提高能源交易的可靠性和完整性。通过分布式信任机制，在不需要信任机构的情况下也能防止篡改和修订，实现可信地记录实时净负载，提高对电网通信的保真度和安全性。

1. 传统系统中程序控制是中心化的，这个专利通过使用智能合约，避免了中心化的控制，执行程序进行价值交换的双方可以相互验证，避免了受到中心化控制和作弊的可能。另外智能合约通过允许能源消费者和采购商相互出售而不是通过多层系统进行交易（在多层系统中，配电和输电系统运营商、电力生产商和供应商在不同层次进行交易，存在中间商差价），从而促进对等能源交换，有效地提高了能源供应效率和降低交易成本。

1. 传统系统中控制程序的逻辑是固定的。这个专利中使用了 人工智能算法自动调整逻辑。
1.1 物联网系统难以避免需要与物理世界交互，在做出决策时就需要更高的智能和现实世界知识。人工智能系统将来自各种传感器的信息转换成智能合约可以执行的代码。例如，关键能源基础设施的所有者和运营商需要针对网络攻击和有害天气条件的保险合同，而通过人工智能生成智能合约则需要可以确定何时触发支付事件。

1.2 将人工智能和区块链功能结合起来，还可以对未经授权试图更改重要数据、配置、应用程序以及网络设备和传感器基础设施的行为提供实时安全响应。自动检测数据异常并通过统一的事件分析时间线减少标准化证据的负担。

1. 传统加密方案通常使用公钥基础设施（public key infrastructure, PKI）解决方案。但该方案通常成本高昂，无法扩展以实现物联网环境的加密要求。比如说，物联网系统环境缺乏必要的计算机处理能力来支持 PKI 的部署。此外，在 PKI 中，通常只有一个颁发机构可以颁发和吊销安全证书。如果此授权受到攻击并且证书被篡改，则其所有用户都可能受到网络攻击。该发明使用了区块链无钥匙签名基础设施 (keyless signature infrastructure, KSI)，通过 KSI 使用数学算法进行身份验证，无需信任密钥或凭证，帮助维护数据交换和其他数字交易的完整性。能够实时大规模地验证物联网数据，提供不变的交易数据，避免了 PKI 存在的问题。KSI 通过使用加密散列函数，证明物联网设备没有变化，防止敏感物联网数据的泄露，并提供了可以证明的加密数据。


#### 具体实施方式

这项专利，使用了区块链分布式账本作为数据库记录数据，通过无钥匙签名基础设施 (keyless signature infrastructure, KSI) 来进行加密，实现物联网数据的可信记录。然后，利用搭载人工智能友好型芯片的设备，进行数据采集检测和未授权的异常检测。采集到的数据再利用人工智能设备，运行机器学习算法自动生成高效的、贴合用户需求的智能合约。通过智能合约来避免中心化的程序控制，实现灵活的、避免中心化作恶的控制。并通过智能合约促进对等能源交换，提高能源供应效率和降低交易成本。支持人工智能的区块链使关键能源交付系统能够越来越自动化以响应自然发生的天气事件、网络或网络物理混合攻击的潜力，从而使一些关键能源基础设施功能增强了自我修复能力和弹性。

#### 总结
这个专利结合人工智能、区块链和智能合约，打造一个运行在区块链上的、可信的人工智能智能合约，适合在人工智能硬件物联网上高效运行，实现了一个构建在电网和物联网等多个复杂系统之上的系统，对能源和物联网进行自动化和现代化，将能源基础设施发展为日益自动化、分布式、清洁和弹性的系统，帮助经济和社会稳健发展。



#### 附图说明

---

<!-- Blockchain is defined as a distributed data base or digital ledger that records transactions of value using a cryptographic signature that is inherently resistant to modification (Tapscott 2016). Combining blockchain based smart contracts with machine learning algorithms presents an opportunity to increase the speed, scale, security and autonomy of complex, distributed internet of things (IoT) environments. Certainly, the need for third parties in executing a transaction will be reduced or even be replaced when an autonomous smart contract can execute and exchange value and services via an autonomous agent. But who will be held responsible when there is an error or when the contract is not successfully executed? While the data and exchange of value captured in blockchain might be immutable, or at least very hard to manipulate, what if the algorithm that establishes the terms of the contract executed is written by a AI agent. What additional challenges and potential solutions should be explored to via AI enabled blockchain solutions to distribute and automate IoT in a more secure way?

基于区块链的智能合约与机器学习算法相结合，提供了一个提高复杂、分布式物联网（IOT）环境的速度、规模、安全性和自主性的机会。当然，当自主智能合约可以通过自主代理执行和交换价值和服务时，第三方执行交易的需求将会减少，甚至会被取代。但如果出现错误或合同执行不成功，谁将承担责任？虽然区块链中捕获的数据和价值交换可能是不可变的，或者至少很难操作，但是如果确定执行的合同条款的算法是由人工智能代理编写的呢？应通过支持人工智能的区块链解决方案探索哪些额外的挑战和潜在的解决方案，以更安全的方式分发和自动化物联网？


This paper explores these questions through an innovative blockchain smart contract application to electricity infrastructure and the array of networked things that are increasing connected responsible for energy generation, transmission, distribution and consumption. This use case highlights how AI enabled blockchain solutions may help increase cyber resilience and optimize complex exchanges of distributed energy resources by encrypting, monitoring and automating transactions and removing third parties. With billions of IoT devices sensing and exchanging information, AI enabled blockchain solutions could also help better analyze data sets from thousands of variables (industrial control system anomalies, frequency, load and voltage changes) and organize them into weighted relationships, which could be tracked through a next generation blockchain solution. As data patterns in these variables are better understood via machine learning computer-based neural networks, the smart blockchain contract could be updated to better secure and exchange energy data.

本文通过一个创新的区块链智能合约应用到电力基础设施，以及一系列网络化的事物来探讨这些问题，这些事物正日益成为产生、传输、分配和消耗能源的连接体。这个用例强调了人工智能区块链解决方案如何通过加密、监控和自动化交易以及删除第三方来帮助提高网络弹性和优化分布式能源资源的复杂交换。由于数十亿物联网设备感知和交换信息，支持人工智能的区块链解决方案还可以帮助更好地分析来自数千个变量（工业控制系统异常、频率、负载和电压变化）的数据集，并将它们组织成加权关系，可通过下一代区块链跟踪这些关系。洗脱。由于这些变量中的数据模式可以通过机器学习计算机神经网络更好地理解，智能区块链合同可以更新，以更好地保护和交换能源数据。

The AI Enabled Blockchain Opportunity: The Evolution of Public Key Infrastructure Encryption

Most cyber security solutions increase cost and reduce functionality in the name of integrity, confidentiality and availability. Blockchain solutions may prove to be an exception in that some applications can improve security and optimize the ability to exchange and track value. Indeed, some blockchain solutions add a layer of cryptography to track digital transactions, but many cyber security challenges remain for securing complex IoT environments. For one, IoT environments are often designed with functionality and cost in mind and security is often an afterthought. IoT often lacks encryption, basic patch management, uses default passwords and communicates in plain text. Poor source code, vulnerable design and improper configuration have also led to several major cyber incidents.

大多数网络安全解决方案都以完整性、机密性和可用性的名义增加了成本并减少了功能。区块链解决方案可能是一个例外，因为一些应用程序可以提高安全性，优化交换和跟踪价值的能力。事实上，一些区块链解决方案增加了一层加密技术来跟踪数字交易，但对于保护复杂的物联网环境，许多网络安全挑战仍然存在。首先，物联网环境的设计通常考虑到功能和成本，而安全性通常是事后诸葛亮。物联网通常缺乏加密、基本补丁管理、使用默认密码和纯文本通信。糟糕的源代码、脆弱的设计和不正确的配置也导致了一些重大的网络事件。

Another challenge with securing IoT from emerging cyber threats, is that public key infrastructure (PKI) solutions are often cost prohibitive and not scalable to realize the encryption requirements of IoT environments. Moreover, legacy systems and converged information technology (IT) and operational technology (OT) environments lack the necessary computer processing power to support some deployments of PKI. This is often seen with analogue equipment in substations and other critical infrastructures. Moreover, with PKI there is often a single authority that both issues and revokes the security certificate. If this authority is attacked and certificate is manipulated, all its users will potentially be vulnerable to cyber-attacks.

保护物联网免受新出现的网络威胁的另一个挑战是，公钥基础设施（PKI）解决方案通常成本高昂，无法扩展以实现物联网环境的加密要求。此外，传统系统和融合信息技术（IT）和操作技术（OT）环境缺乏必要的计算机处理能力来支持PKI的某些部署。在变电站和其他关键基础设施中，模拟设备经常会出现这种情况。此外，在pki中，通常只有一个颁发机构可以颁发和吊销安全证书。如果此授权受到攻击并且证书被篡改，则其所有用户都可能受到网络攻击。

Thus, PKI must continue to evolve to secure IoT environments or a better solution needs to be scaled up. Blockchain keyless signature infrastructure KSI presents a potential path forward. KSI is a promising solution patented by Gaurdtime, one of the largest blockchain providers by revenue, which helps preserve the integrity of data exchanges and other digital transactions using a mathematical algorithm for authentication without the need for trusted keys or credentials. KSI authenticates IoT data at scale, in real time, providing immutable transaction data without several of the challenges of PKI. The following image further describes KSI’s cryptographic hash function, highlight how the hash function can help prove the IoT device hasn’t change, preventing the disclosure of sensitive IoT data and providing cryptographic proof that can be proven.

因此，pki必须继续发展以确保物联网环境的安全，或者需要扩大一个更好的解决方案。区块链无钥匙签名基础设施KSI提供了一条潜在的前进之路。KSI是Gaurdtime（按收入计算最大的区块链提供商之一）专利的一个有前途的解决方案，它使用数学算法进行身份验证，无需信任密钥或凭证，帮助维护数据交换和其他数字交易的完整性。KSI实时大规模地验证物联网数据，提供不变的交易数据，而不存在PKI的几个挑战。下图进一步描述了ksi的加密散列函数，重点介绍了散列函数如何帮助证明物联网设备没有变化，防止敏感物联网数据的泄露，并提供了可以证明的加密证明。

Researchers at Pacific Northwest National Laboratory, Guardtime Guardtime, the United States Department of Energy (DOE), Washington State University, Tennessee Valley Authority (TVA), Siemens and the Department of Defense Homeland Defense and Security Information Analysis Center (HDIAC) are developing a KSI enabled blockchain solution to help secure distributed energy IoT environments found in modern electricity infrastructure. This is especially important because as we modernize our energy infrastructure, the speed, size and complexity of energy data and transactions exchanged increases exponentially.

太平洋西北国家实验室（Pacific Northwest National Laboratory）、Guardtime Guardtime、美国能源部（DOE）、华盛顿州立大学、田纳西河谷管理局（TVA）、西门子（Siemens）和国防部国土防卫与安全信息分析中心（HDIAC）的研究人员正在开发一个支持KSI的区块链解决方案，以帮助安全现代电力基础设施中的再分布能源物联网环境。这一点尤为重要，因为随着我们能源基础设施的现代化，能源数据和交易交换的速度、规模和复杂性呈指数级增长。

To help overcome these challenges, blockchain keyless signature infrastructure technology provides a unique value proposition in its potential to help optimize and secure these critical data sets from emerging cyber threats. AI enabled blockchain shows potential to enable critical energy delivery systems to be increasingly automated to respond to a naturally occurring weather event, cyber or cyber-physical hybrid attack, in a way that that some critical energy infrastructure functions become increasingly self-healing and resilient.

为了帮助克服这些挑战，区块链无键签名基础设施技术在帮助优化和保护这些关键数据集免受新出现的网络威胁方面提供了独特的价值主张。支持人工智能的区块链显示了使关键能源交付系统能够越来越自动化以响应自然发生的天气事件、网络或网络物理混合攻击的潜力，从而使一些关键能源基础设施功能变得越来越自我修复和有弹性。

Blockchain’s digital ledger and cryptography signed transaction data may help increase the trustworthiness and integrity energy transactions. Combined with machine learning and AI enabled energy delivery systems, these systems may also have more control and flexibility in automating, monitoring and auditing of complex energy exchanges at the grid’s edge.


区块链的数字账本和加密签名交易数据可能有助于提高能源交易的可靠性和完整性。结合机器学习和人工智能的能量传递系统，这些系统在电网边缘复杂能量交换的自动化、监控和审计方面也可能具有更多的控制和灵活性。

Combining AI and blockchain capability could also provide a real-time security response to unauthorized attempts to change critical EDS data, configurations, applications, and network appliance and sensor infrastructure. Autonomous detection of data anomalies and reduces burden with normalized evidence across a unified timeline for incident analysis. A data exchange platform using smart contracts for the automated trading and settlement of contracts in the electricity production value chain.

将人工智能和区块链功能结合起来，还可以对未经授权试图更改重要EDS数据、配置、应用程序以及网络设备和传感器基础设施的行为提供实时安全响应。自动检测数据异常并通过统一的事件分析时间线减少标准化证据的负担。一个使用智能合约的数据交换平台，用于电力生产价值链中合约的自动交易和结算。

Blockchain is defined as a distributed data base or digitalledger that records transactions of value using a cryptographic signature that is inherently resistant to modification (Tapscott 2016). Blockchain is a distributed database that maintains a continuously growing list of records, called blocks, secured from tampering and revision. Each block contains a timestamp and a link to a previous block. Blockchain-based smart contracts can be executed without human interaction (Franco 2014) and the data is more resistant to modification as the data in a block cannot be altered retroactively. Blockchain smart contracts are defined as technologies or applications that exchange value without intermediaries acting as arbiters of money and information (Tapscott 2016).

区块链被定义为一个分布式数据库或数字账本，它使用固有的抗修改的加密签名记录价值交易（Tapscott 2016）。区块链是一个分布式数据库，它维护一个不断增长的记录列表，称为块，以防篡改和修订。每个块包含一个时间戳和到上一个块的链接。基于区块链的智能合约可以在没有人工交互的情况下执行（Franco 2014），数据更难修改，因为块中的数据不能进行追溯性修改。区块链智能合约被定义为在没有中介机构充当货币和信息仲裁者的情况下交换价值的技术或应用程序（Tapscott 2016）。

A keyless signature blockchain infrastructure (KSBI) differs from proof or work blockchain based crypto currencies as it is based around a concept of permission- based blockchain - to provide widely witnessed evidence on what can be considered the truth, independently of any single party and while retaining complete confidentiality of the original data. Another unique characteristic that differentiates the KSBI from other distributed ledger solutions are its ability to scale to industrial applications to add one trillion data items to the blockchain every second, and to verify the data item from the blockchain within the next second. The ability to transact data at sub second speeds is essential to handle the increasing data requirements of a modern power grid (Mylrea et al. 2017).

无钥匙签名区块链基础设施（ksbi）不同于基于证明或工作区块链的加密货币，因为它基于基于许可的区块链概念，以提供可被视为真理的广泛证据，独立于任何一方，同时保留原始数据的完全保密性。数据。另一个区别于其他分布式账本解决方案的独特特征是，它能够扩展到工业应用程序，每秒向区块链添加1万亿个数据项，并在下一秒钟内从区块链验证数据项。在亚秒级速度下处理数据的能力对于处理现代电网日益增长的数据需求至关重要（Mylrea等人2017）。

KSBI is based on Guardtime’s patented technology keyless signature infrastructure (KSI_® which has been in production use since 2007, is employed by various world’s governments – i.e. Estonia and Defense primes in United States - and is beginning to see adoption in the private sector for application for their systems and networks. A KSBI may also help realize several cybersecurity and compliance goals for the energy sector, such as: 

KSBI基于GuardTime的专利技术无钥匙签名基础设施（KSI_xAE；自2007年开始投入生产使用，被世界各国政府（即爱沙尼亚和美国的国防总理）采用，并开始在私营部门采用，以应用其系统和网络。KSBI还可以帮助实现能源部门的几个网络安全和合规目标，例如：

Smart contracts: Smart contracts execute and record transaction in the blockchain load ledger through blockchain enabled advanced metering infrastructure (AMI). Blockchain based smart contracts may help facilitate consumer level exchange of excess generation from DER. This could provide additional storage and help substation load balancing from bulk energy systems. Moreover, smart contract data is secured in part through decentralized storage of all transactions of energy flows and business activities (Mylrea et al. 2017). 

智能合约：智能合约通过支持区块链的高级计量基础设施（AMI）在区块链负载分类账中执行和记录交易。基于区块链的智能合约可能有助于促进消费者级别的订单超额发电交换。这可以提供额外的存储，并帮助从大容量能源系统的变电站负载平衡。此外，智能合约数据部分通过分散存储所有能源流和业务活动的交易而得到保护（Mylrea等人2017）。

Secure Data Storage in Cryptographically Signed Distributed Ledger: Blockchain can help fill various optimization and security gaps and improve the state of the art in grid resilience by providing an atomically verifiable cryptographic signed distributed ledger to increase the trustworthiness, integrity and security of energy delivery systems at the edge. Blockchain can be used to verify time, user, transaction data and protect this data with an immutable crypto signed distributed ledger (Mylrea et al. 2017).

加密签名分布式账本中的安全数据存储：区块链通过提供一个原子可验证的加密签名分布式账本，可以帮助填补各种优化和安全缺口，提高电网恢复能力的最新水平，从而提高在T T的能源输送系统的可靠性、完整性和安全性。他有优势。区块链可用于验证时间、用户、交易数据，并使用不变的加密签名分布式账本保护这些数据（Mylrea等人2017）。

Certainly, AI enabled blockchain will be disruptive and replace jobs, especially traditional 3rd parties that are replaced by new consensus algorithms and distributed trust mechanisms. Energy aggregators and meter readers could potentially be replaced by a dynamic distributed ledger. Blockchain innovation will also create new energy jobs, value, and markets. Even as technology empowers humans, it also changes the relationship between man and machine, technology and organizations, society and innovation. Autonomous blockchain organizations may distribute power and leadership via cryptographic votes that establish equity against a contract or even mission statement. For example, future energy organizations may have stakeholders govern what type of energy mix they would like and have that preference or willingness to pay be capture in a smart contract. Blockchain AI empowered energy organizations might be increasingly autonomous made up decentralized contractors and investors with power to vote, invest and delivery services based on an immutable smart contract that captures who, what, when and where services are executed and shared in a transparent immutable ledger.



当然，支持人工智能的区块链将具有破坏性并取代工作岗位，尤其是传统的第三方，这些第三方将被新的共识算法和分布式信任机制所取代。能量聚合器和抄表器可能会被动态分布式账本所取代。区块链创新还将创造新的能源就业机会、价值和市场。即使技术赋予人类力量，它也改变了人与机器、技术与组织、社会与创新之间的关系。自治区块链组织可以通过加密投票来分配权力和领导权，这些投票建立了对合同甚至任务声明的公平性。例如，未来的能源组织可能会让利益相关者控制他们想要的能源组合类型，并在智能合约中获得这种偏好或支付意愿。区块链人工智能授权的能源组织可能越来越具有自主性，由分散的承包商和投资者组成，他们有权根据一个不变的智能合约来投票、投资和交付服务，该智能合约在透明不变的分类账中记录服务的执行者、执行者、执行者、时间和地点。

The notion of a “self-bootstrapped” organizations with crypto equities leveraging independently contractors guided by decentralized blockchain voting has been explored (Levine 2014). Bit congress has established a blockchain based voting system. The country of Georgia is leveraging blockchain to facilitate real estate licensing. Estonia has established a privacy preserving secure virtual government using keyless signature infrastructure blockchain. These examples highlight how technology can help distribute trust and reduce redundancy in everything from billing to middle management, creating new value for organizations in an increasingly decentralized autonomous society. Reducing redundancy creates new value and more competitive organizations (Lawless 2017)



已经探讨了“自启动”组织的概念，即在分散的区块链投票的指导下，利用独立承包商的加密股票（Levine 2014）。比特大会建立了一个基于区块链的投票系统。格鲁吉亚国家正在利用区块链促进房地产许可。爱沙尼亚利用无钥匙签名基础设施区块链建立了一个保护隐私的安全虚拟政府。这些例子强调了技术如何帮助分配信任，减少从计费到中间管理的所有方面的冗余，为日益分散的自治社会中的组织创造新的价值。减少冗余创造了新的价值和更具竞争力的组织（Lawless 2017）


Blockchain and AI integration and innovation may present a more resilient and efficient path for decentralized cyber and physical devices to interactive, transforming modern infrastructure into array of smart autonomous systems of systems. Increased autonomy and control is essential to optimized the rapidly growing “Internet of Things” environment that Gartner has predicted to include 26 billion devices by 2020 (Gartner 2013).

区块链和人工智能集成与创新可能为分散的网络和物理设备提供一条更具弹性和效率的互动路径，将现代基础设施转变为一系列智能自主系统。增强自主性和控制对于优化快速增长的“物联网”环境至关重要，Gartner预计到2020年将包括260亿台设备（Gartner 2013年）。

“Simple and easy to write contracts appear to be sufficient for many entirely digital transactions. But as these systems start to interact with the physical world, there is likely to be a need for greater intelligence and real world knowledge in making decisions. AI systems will be needed to translate information from a wide variety of sensors into precise terms that smart contracts can act upon. In the other direction, contracts that lead to physical actions (such as delivery of items) will need to interface with human and robotic agents. For example, owner and operators of critical energy infrastructure might want insurance contracts against cyber-attacks and harmful weather conditions and a smart contract would need to determine when the payout event is triggered (Levine 2014).



“对于许多完全数字化的交易来说，简单易写的合同似乎已经足够了。但是，随着这些系统开始与物理世界交互，在做出决策时可能需要更高的智能和现实世界知识。人工智能系统将需要将来自各种传感器的信息转换成智能合约可以执行的精确术语。在另一个方向上，导致物理行为（如物品交付）的合同将需要与人类和机器人代理交互。例如，关键能源基础设施的所有者和运营商可能需要针对网络攻击和有害天气条件的保险合同，而智能合同则需要确定何时触发支付事件（Levine 2014）。

These grid optimization, automation and resilience improvements are essential operations and design criteria as we modernize our power grid. However, cybersecurity is often an afterthought as vendors and end users prioritize functionality and cost, leaving our power grid, the backbone of our economy, potentially vulnerable to a cyber-attack. This is especially true at the grid’s edge which continues to increase the size and speed of data being collected and exchanged in absence of clear cybersecurity and IoT standards and regulation. Thus, the grid lacks the necessary defenses to prevent disruption and manipulation of DERs, grid edge devices and associated electricity infrastructure. Moreover, as the smart grid increases its connectivity and communications with buildings, cyber vulnerabilities will extend behind the meter into “smart” buildings, which also have a host of documented cybersecurity vulnerabilities.

随着电网现代化，这些电网优化、自动化和恢复能力的改进是必不可少的操作和设计标准。然而，网络安全往往是在供应商和最终用户优先考虑功能和成本后才考虑的问题，使我们的电网（我们经济的支柱）可能容易受到网络攻击。这在网格的边缘尤其如此，在缺乏明确的网络安全和物联网标准和法规的情况下，数据收集和交换的规模和速度将继续增加。因此，电网缺乏必要的防御措施，以防止破坏和操纵配电系统、电网边缘设备和相关电力基础设施。此外，随着智能电网增加其与建筑物的连接和通信，网络漏洞将延伸到仪表后面的“智能”建筑物，这些建筑物也有大量记录在案的网络安全漏洞。

Blockchain technology can also be applied to the smart grid to help reduce costs by cutting out 3rd parties and increasing the arbitrage opportunity for individuals to produce and sell energy to each other. Smart contracts facilitate peer-to-peer energy exchanges by enabling energy consumers and procures to sell to each other, instead of transacting through a multi-tiered system, in which distribution and transmission system operators, power producers, and suppliers transact on various levels (Mylrea and Gourisetti 2017). In April 2016, one of the first use cases was demonstrated where energy generated in a decentralized fashion was sold directly between neighbors in New York via a blockchain system, demonstrating that energy producers and energy consumers could execute energy supply contracts without involving a third-party intermediary; effectively increasing speed and reducing costs of the transaction (PWC 2017).


区块链技术也可以应用于智能电网，通过切断第三方并增加个人相互生产和销售能源的套利机会来帮助降低成本。智能合约通过允许能源消费者和采购商相互出售而不是通过多层系统进行交易，从而促进对等能源交换，在多层系统中，配电和输电系统运营商、电力生产商和供应商在不同层次进行交易（Mylrea和Gourisetti，2017年）。2016年4月，第一个使用案例中的一个通过区块链系统在纽约邻居们之间直接销售以分散方式产生的能源，证明能源生产商和能源消费者可以在不涉及第三方中介的情况下执行能源供应合同；有效地提高了能源供应效率。ED和降低交易成本（普华永道2017年）。



In addition to potential cost savings, transaction data might be more secure through decentralized storage and multifactor verification of transactions in the blockchain distributed ledger (PWC 2017). Blockchain reduces the need for 3rd parties to process transactions: Electricity is generated  Consumer buys the electricity  blockchain based meters update the blockchain, creating a unique timestamped block for verification in a distributed ledger: 1) At the distribution level, system operators can leverage the blockchain to receive energy transaction data to charge their network costs to consumers; 2) Reduces data requirements and increases speed of clearing transactions for transmission system operators as transactions could be executed and settled on the basis of actual consumption (Mylrea and Gourisetti 2017).


除了潜在的成本节约之外，通过分散存储和区块链分布式账本中交易的多因素验证，交易数据可能更安全（pwc 2017）。区块链减少了第三方处理交易的需要：发电消费者购买基于电力区块链的电表更新区块链，在分布式账本中创建唯一的时间戳块进行验证：1）在分发级别，系统运营商可以利用区块链接收能量传输行动数据向用户收取其网络成本；2）减少数据需求，提高传输系统运营商的结算交易速度，因为交易可以在实际消费的基础上执行和结算（Mylrea和Gourisetti，2017年）。

Smart contracts execute and record transaction in the blockchain load ledger through blockchain enabled advanced metering infrastructure (AMI). Blockchain based smart contracts can facilitate consumer level exchange of excess generation from DERs, EVs, etc. This could provide additional storage and help substation load balancing from bulk energy systems. Moreover, smart contract data is secured in part through decentralized storage of all transactions of energy flows and business activities. This highlights the disruptive potential for blockchain on energy markets through the introduction of a more autonomous and decentralized transaction model.

智能合约通过支持区块链的高级计量基础设施（AMI）在区块链负载分类账中执行和记录交易。基于区块链的智能合约可以促进用户级交换来自ders、evs等的过剩发电量。这可以提供额外的存储，并帮助从大容量能源系统实现变电站负载平衡。此外，智能合约数据部分是通过分散存储所有能量流和业务活动的交易而得到保护的。这突出了通过引入更自主和分散的交易模型，在能源市场上区块链的破坏潜力。

This peer to peer system may reduce or even replace the need for a meter operator if the meter blockchain is shared with the distribution system operator.

如果电表区块链与配电系统运营商共享，这种点对点系统可能减少甚至取代对电表运营商的需求。

Currently, the power grid lacks the necessary security and resilience to prevent cyber-attacks on DERs, grid edge devices and associated electricity infrastructure. Cyber vulnerabilities and interoperability challenges also extend behind the meter into building automation and controls systems. Applying blockchain could help increase fidelity and security of buildings to grid communications. Moreover, multiple customers can leverage the same widely witnessed blockchain to cryptographically verify the other entities data when needed, creating a distributed trust mechanism. Blockchain may also help solve several optimization and reliability challenges that have been ushered in with grid modernization.

目前，电网缺乏必要的安全性和恢复能力，无法防止对灾害应急响应系统、电网边缘设备和相关电力基础设施的网络攻击。网络脆弱性和互操作性挑战也延伸到了建筑自动化和控制系统的后面。应用区块链有助于提高建筑物对电网通信的保真度和安全性。此外，多个客户可以在需要时利用同一广泛见证的区块链加密验证其他实体数据，从而创建分布式信任机制。区块链还可以帮助解决网格现代化带来的几个优化和可靠性挑战。

Currently, time-lags for payment and uncollected bills leaves value on the table and the real cost associated with the energy value chain is not captured. Blockchain can record real time net loads and smart contracts execute customers distributed generated sales and purchases. Currently, grid operators lack visibility and control of real- time power flows and injections from DERs and distributed generation customers. Blockchain can help optimize network data and record residual energy at the substation level. Increasing the fidelity and control of utility data will also help settle with bulk systems as well as negotiate future contracts.

目前，支付和未收回票据的时间滞后会使价值留在表中，而与能源价值链相关的实际成本并未被捕获。区块链可以记录实时净负载，智能合约执行客户分布式生成的销售和购买。目前，电网运营商缺乏对来自分布式发电客户和分布式发电客户的实时电力流和注入的可视性和控制。区块链可以帮助优化网络数据并记录变电站级别的剩余能量。提高公用事业数据的保真度和控制也将有助于解决批量系统以及谈判未来的合同。

Blockchain, AI and IoT have a lot of buzz right now. Reading the news one might assume that blockchain is a panacea for all that ills us – climate change, cyber security, volatile financial systems. AI articles suggest that robots are coming and may take our jobs. Internet of Things or IoT cyber incidents remind us that everything is increasingly connected to the internet and collecting and exchanging data that is potentially vulnerable. While these are disruptive in their own way and create some exciting new opportunities, many challenges remain. Several fundamental policy, regulatory and scientific challenges remain before blockchain realizes its disruptive potential. This sections explores some of the challenges as they relate to block chain’s application to the array of things.

区块链、人工智能和物联网现在很流行。在阅读新闻时，人们可能会认为区块链是解决我们所有问题的灵丹妙药——气候变化、网络安全、动荡的金融系统。人工智能的文章表明机器人正在到来，可能会夺走我们的工作。物联网或物联网网络事件提醒我们，所有事物都越来越多地与互联网连接，并收集和交换可能易受攻击的数据。虽然这些都是以他们自己的方式破坏和创造一些令人兴奋的新机会，许多挑战仍然存在。在区块链实现其颠覆性潜力之前，仍存在一些基本政策、监管和科学挑战。本节将探讨与区块链应用于事物数组相关的一些挑战。

Applying AI Blockchain to modernize electricity infrastructure also requires speed, agility and affordable technology. AI enhanced algorithms are not always cheap and often require prodigious data sets that must be broken down into a code that makes sense. However, there is a lot of noise or distracting data being exchanged in electricity infrastructure, making it difficult to identify what caused an anomaly – what is a software hire, cyber-attack, weather event, all the above? It can be very difficult to determine what normal looks like. Thus, developing an AI enhanced grid requires breaking down the data into observable patterns, which is also very challenging from a cyber perspective as threats are complex, non-linear and evolving.

将人工智能区块链应用于电力基础设施的现代化还需要速度、灵活性和经济实惠的技术。人工智能增强算法并不总是便宜的，而且常常需要大量的数据集，这些数据集必须分解成有意义的代码。然而，电力基础设施中存在大量的噪音或干扰数据，使得很难确定造成异常的原因——什么是软件租用、网络攻击、天气事件等等？很难确定正常情况。因此，开发一个人工智能增强的网格需要将数据分解成可观察的模式，从网络的角度来看，这也是非常具有挑战性的，因为威胁是复杂的、非线性的和不断演变的。

New blockchain opportunities are also accompanied by the lack of policy, legal and regulatory frameworks. For example, even if some intermediaries are replaced in the energy sector, there still needs to be schedule and forecast submitted to the transmission system operator for electricity infrastructure to be reliable. Another challenge is incorporating individual blockchain consumers into a balancing group and having them comply with market reliability and requirements and submit accurate demand forecasts to the network operator. Managing a balancing group is not a trivial task and could potentially increase costs of managing the blockchain. To avoid costly disruptions, blockchain autonomous data exchanges, such as demand forecasts from the consumer to the network operator will need to be stress tested for security and reliability before deployed at scale.

新的区块链机会也伴随着缺乏政策、法律和监管框架。例如，即使能源部门更换了一些中间商，仍需要向输电系统运营商提交时间表和预测，以确保电力基础设施的可靠性。另一个挑战是将单个区块链消费者纳入一个平衡小组，让他们遵守市场可靠性和要求，并向网络运营商提交准确的需求预测。管理一个平衡组不是一项简单的任务，可能会增加管理区块链的成本。为了避免代价高昂的中断，区块链自主数据交换，例如从消费者到网络运营商的需求预测，在大规模部署之前，需要对安全性和可靠性进行压力测试。

Applying blockchain to modernizing and secure electricity infrastructure also presents several cyber security challenges. For example, Ethereum based smart contracts provide the ability for anyone to write electronic code that can be executed on a blockchain. For example, an energy producer or consumer agrees to buy or sell renewable energy from a neighbor for an agreed upon price that is captured in blockchain based smart contract. AI could help increase the efficiency and automate the auction to include other bidders and sellers in a more efficient and dynamic way, but this would require a lot more data and analysis of that data to recognize discernable patter in that data to inform the AI algorithm of the smart contract.

将区块链应用于现代化和安全的电力基础设施也带来了一些网络安全挑战。例如，基于以太坊的智能合约为任何人提供了编写可在区块链上执行的电子代码的能力。例如，能源生产商或消费者同意以基于区块链的智能合约中捕获的约定价格从邻居处购买或销售可再生能源。人工智能可以帮助提高效率和自动拍卖，以更有效和动态的方式包括其他竞买人和卖家，但这将需要更多的数据和数据分析，以识别该数据中可识别的模式，以通知智能合约的人工智能算法。

This also requires the code of the blockchain to be more resilient to cyber-attacks. Previously, Ethereum has shown to have several vulnerabilities that may underline the trustworthiness of this transaction mechanism. Vulnerabilities in the code have been exploited in at least three multi-million dollar cyber incidents. In June 2016, DAO, was hacked exploiting vulnerable smart contract code and extracting approximately $50 million dollars. In July 2017, vulnerable code in am Ethereum wallet was exploited to extract $30 million dollars of cryptocurrency. In January 2018, hackers stole roughly 58 billion yen ($532.6 million) from a Tokyo-based cryptocurrency exchange. Coincheck Inc. This incident highlighted the need for increased security and regulatory protection for cryptocurrencies and other blockchain applications. The Coincheck hack appears to have exploited vulnerabilities in a “hot wallet” which is a crypto currency wallet that is connected to the internet. In contrast, cold wallets, such as Trezor and Ledger Nano S, are cryptocurrency wallets that are stored offline.

这还要求区块链的代码对网络攻击更具弹性。以前，以太坊已经显示出了一些漏洞，这些漏洞可能突出了该交易机制的可靠性。该代码中的漏洞已在至少300万美元的网络事件中被利用。2016年6月，DAO被黑客攻击，利用易受攻击的智能合约代码，并获取了大约5000万美元。2017年7月，AM以太坊钱包中的易受攻击代码被用于提取3000万美元的加密货币。2018年1月，黑客从东京的加密货币交易所窃取了大约580亿日元（5.326亿美元）。Coincheck Inc.这一事件凸显了加密货币和其他区块链应用程序需要加强安全和监管保护。Coincheck黑客似乎利用了“热钱包”中的漏洞，这是一个连接到互联网的加密货币钱包。相比之下，Trezor和Ledger Nano等冷钱包是离线存储的加密货币钱包。

Despite being a centralized currency, Coincheck was a centralized cryptocurrency exchange with a single point of failure. However, the blockchain shared ledger of the account may potentially be able to tag and follow the stolen coins and identify any account which receives them (Fadilpašić and Garlick 2018). Storing prodigious data sets that constantly growing on a blockchain can also create potential latency or bloat in the chain, requiring large amounts of ram and memory on a server. These requirements for ethereum based smart contracts have grown over time and the block takes a longer time to get processed. For time, sensitive energy transactions this may create speed, scale and cost issues of the smart contract is not designed properly.

尽管Coincheck是一种集中的货币，但它是一种单一故障点的集中加密货币交换。然而，该账户的区块链共享分类账可能能够标记和跟踪被盗硬币，并识别收到它们的任何账户（Fadilpa_i_和Garlick 2018）。存储在区块链上不断增长的庞大数据集也可能在链中产生潜在的延迟或膨胀，需要在服务器上使用大量的RAM和内存。基于以太坊的智能合约的这些需求随着时间的推移而增长，并且处理该区块需要更长的时间。对于时间而言，敏感的能源交易这可能会导致智能合约的速度、规模和成本问题设计不当。

 -->

<!-- 





区块链技术是一种去中心化的分布式数据库技术，具有去中心化、公开透明、不可篡改、可信任等多种特点，适用于诸多对数据可靠性具有高需求的应用场景。 智能合约是执行在区块链上层的脚本，具有图灵完备与高可塑性的特性。使用者可以将复杂的商业逻辑以智能合约的模式实现，通过将智能合约交由区块链底层托管的模式，将需要多方信任才可完成的商业模式变成无需信任化。

智能合约做到无需的信任的前置条件是将与合约相关的数据存储到区块链上，从而做到合约数据不可被篡改。但区块链是去中心化数据库的本质决定了其难以像传统的数据库一样对单个应用存储大量的数据，从而限制了智能难以承载拥有海量数据的大型应用。



发明内容
为解决上述问题，本说明书一个或多个实例提供技术方案如下：
现阶段的智能合约通过将数据可持续化存储在智能合约的KV存储空间上来支撑智能合约的运行，智能合约的KV存储空间上限限制了智能合约可以处理数据的深度。为解决上述问题，本说明书一个或多个实例提供技术方案如下：将智能合约的大量数据存储从智能合约的存储空间中剥离出来，存储在任意类型的链下的数据库中。在智能合约的层面上，为智能合约添加外部数据验证模块用来保证智能合约不会处理不可信的外部数据。数据验证模块的核心功能是使用有限的存储资源映射外部大量数据集的一个状态，并为智能合约提供三个支持功能：第一是在有新数据插入智能合约外部数据库的时候做一个映射状态的更新，保证数据验证模块与链下数据库的同步；第二是在智能合约调用外部数据集的时候验证数据的真伪性；第三是在数据从外部数据库中删除的时候，做一个映射状态的更新，保证数据验证模块与链下数据库的同步。通过这三个功能使得数据验证模块成为联通链上智能合约与链下外部数据库的中间键，保证智能合约的外部存储状态与外部数据库的存储数据一直并且可以无需信任的调用任何外部数据库。
数据验证层的创新设计是结合了哈希与二叉树两种算法，使得可以用类似编程中指针的模式，以一个定长的字节来映射无穷大的数据集。每一条链下数据库的数据可以通过哈希算法压缩成一个定长的字节，再将所有的哈希组成一颗二叉树结构。再通过默克尔树这一数据结构的归集模式将所有的哈希压缩成根哈希，用来存储到链上智能合约中的数据验证层来映射链下数据库。

具体实施方式
在智能合约的持续化存储层，添加一个长度为64位的哈希来代表锚定的可信任外部数据库的状态，这个64位的哈希本质意义上上代表了此智能合约映射的外部数据库的默克尔树的根哈希。 在智能合约第一次初始化完成时，设置哈希为64个空字节以表示数据库为空。当有数据需要插入到智能合约之中的时候，链上和链下将分两步完成。对于链下的外部数据库，正常插入数据之后计算数据库的默克尔树根哈希，并将默克尔树的结构变化转换成区块链的交易做上链。 对于智能合约，在检测到有与自身相关的数据库信息上链之后，则会根据上链的交易更新合约自身的64位哈希。由于数据插入信息上链的交易中有包含整体数据库状态变化的默克尔树路径变化，所以智能合约可以用可控数量的哈希验证来保证信息的正确性。当数据库中的信息发生删除的情况时候，也是由链下数据库首先发生改变，在改变之后将默克尔的改动信息发起交易上链。智能合约在检测到有与其对应的交易发生时，通过默克尔书的改变路径验证检验删除的合法性。在智能合约中需要使用某个已存在外部数据库中的数据时，需要提供数据的原生数据与生成当年默克尔数的兄弟节点哈希。通过默克尔树存在证明的数据则可在合约中调用。

总结
一种区块链智能合约锚定可信任外部数据库的办法的核心是利用默克尔树这一数据结构将无限大的数据集压缩成64位长度的哈希来优化智能合约的持续化存储所虚空间。在外部数据的增删改查过程中，利用平衡树的特性保证了智能合约的验证的开销永远保持在一个合理的范围内。利用合理范围内的区块链节点CPU资源换取近乎无上限的可信任链下存储资源。



附图说明
图一是智能合约锚定可信任外部数据库方法的初始化流程

 -->
