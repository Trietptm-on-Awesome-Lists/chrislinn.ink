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
本说明书的一个或多个实例涉及区块链和人工智能计算技术领域，尤其涉及运行在区块链上的、通过人工智能硬件进行计算的智能合约，对能源和物联网进行自动化和现代化的方法

#### 技术背景
区块链赋能的智能合约展现了在不需要第三方信任机构的情况下价值交换的潜力。

_TODO: 拆分_

人工智能，密码学，分布式信任算法和智能合约组合起来能更加安全、高效地来对价值、商品和服务进行交换。这个专利结合人工智能、区块链和智能合约，打造一个运行在区块链上的、可信的人工智能智能合约，适合在人工智能硬件物联网上高效运行，实现了一个构建在电网和物联网等多个复杂系统之上的系统，对能源和物联网进行自动化和现代化，将能源基础设施发展为日益自动化、分布式、清洁和弹性的系统，帮助经济和社会稳健发展。

Blockchain is defined as a distributed data base or digital ledger that records transactions of value using a cryptographic signature that is inherently resistant to modification (Tapscott 2016). Combining blockchain based smart contracts with machine learning algorithms presents an opportunity to increase the speed, scale, security and autonomy of complex, distributed internet of things (IoT) environments. Certainly, the need for third parties in executing a transaction will be reduced or even be replaced when an autonomous smart contract can execute and exchange value and services via an autonomous agent. But who will be held responsible when there is an error or when the contract is not successfully executed? While the data and exchange of value captured in blockchain might be immutable, or at least very hard to manipulate, what if the algorithm that establishes the terms of the contract executed is written by a AI agent. What additional challenges and potential solutions should be explored to via AI enabled blockchain solutions to distribute and automate IoT in a more secure way?

This paper explores these questions through an innovative blockchain smart contract application to electricity infrastructure and the array of networked things that are increasing connected responsible for energy generation, transmission, distribution and consumption. This use case highlights how AI enabled blockchain solutions may help increase cyber resilience and optimize complex exchanges of distributed energy resources by encrypting, monitoring and automating transactions and removing third parties. With billions of IoT devices sensing and exchanging information, AI enabled blockchain solutions could also help better analyze data sets from thousands of variables (industrial control system anomalies, frequency, load and voltage changes) and organize them into weighted relationships, which could be tracked through a next generation blockchain solution. As data patterns in these variables are better understood via machine learning computer-based neural networks, the smart blockchain contract could be updated to better secure and exchange energy data.


The AI Enabled Blockchain Opportunity: The Evolution of Public Key Infrastructure Encryption

Most cyber security solutions increase cost and reduce functionality in the name of integrity, confidentiality and availability. Blockchain solutions may prove to be an exception in that some applications can improve security and optimize the ability to exchange and track value. Indeed, some blockchain solutions add a layer of cryptography to track digital transactions, but many cyber security challenges remain for securing complex IoT environments. For one, IoT environments are often designed with functionality and cost in mind and security is often an afterthought. IoT often lacks encryption, basic patch management, uses default passwords and communicates in plain text. Poor source code, vulnerable design and improper configuration have also led to several major cyber incidents.

Another challenge with securing IoT from emerging cyber threats, is that public key infrastructure (PKI) solutions are often cost prohibitive and not scalable to realize the encryption requirements of IoT environments. Moreover, legacy systems and converged information technology (IT) and operational technology (OT) environments lack the necessary computer processing power to support some deployments of PKI. This is often seen with analogue equipment in substations and other critical infrastructures. Moreover, with PKI there is often a single authority that both issues and revokes the security certificate. If this authority is attacked and certificate is manipulated, all its users will potentially be vulnerable to cyber-attacks.

Thus, PKI must continue to evolve to secure IoT environments or a better solution needs to be scaled up. Blockchain keyless signature infrastructure KSI presents a potential path forward. KSI is a promising solution patented by Gaurdtime, one of the largest blockchain providers by revenue, which helps preserve the integrity of data exchanges and other digital transactions using a mathematical algorithm for authentication without the need for trusted keys or credentials. KSI authenticates IoT data at scale, in real time, providing immutable transaction data without several of the challenges of PKI. The following image further describes KSI’s cryptographic hash function, highlight how the hash function can help prove the IoT device hasn’t change, preventing the disclosure of sensitive IoT data and providing cryptographic proof that can be proven.

Researchers at Pacific Northwest National Laboratory, Guardtime Guardtime, the United States Department of Energy (DOE), Washington State University, Tennessee Valley Authority (TVA), Siemens and the Department of Defense Homeland Defense and Security Information Analysis Center (HDIAC) are developing a KSI enabled blockchain solution to help secure distributed energy IoT environments found in modern electricity infrastructure. This is especially important because as we modernize our energy infrastructure, the speed, size and complexity of energy data and transactions exchanged increases exponentially.

To help overcome these challenges, blockchain keyless signature infrastructure technology provides a unique value proposition in its potential to help optimize and secure these critical data sets from emerging cyber threats. AI enabled blockchain shows potential to enable critical energy delivery systems to be increasingly automated to respond to a naturally occurring weather event, cyber or cyber-physical hybrid attack, in a way that that some critical energy infrastructure functions become increasingly self-healing and resilient.

Blockchain’s digital ledger and cryptography signed transaction data may help increase the trustworthiness and integrity energy transactions. Combined with machine learning and AI enabled energy delivery systems, these systems may also have more control and flexibility in automating, monitoring and auditing of complex energy exchanges at the grid’s edge.

Combining AI and blockchain capability could also provide a real-time security response to unauthorized attempts to change critical EDS data, configurations, applications, and network appliance and sensor infrastructure. Autonomous detection of data anomalies and reduces burden with normalized evidence across a unified timeline for incident analysis. A data exchange platform using smart contracts for the automated trading and settlement of contracts in the electricity production value chain.

Blockchain is defined as a distributed data base or digitalledger that records transactions of value using a cryptographic signature that is inherently resistant to modification (Tapscott 2016). Blockchain is a distributed database that maintains a continuously growing list of records, called blocks, secured from tampering and revision. Each block contains a timestamp and a link to a previous block. Blockchain-based smart contracts can be executed without human interaction (Franco 2014) and the data is more resistant to modification as the data in a block cannot be altered retroactively. Blockchain smart contracts are defined as technologies or applications that exchange value without intermediaries acting as arbiters of money and information (Tapscott 2016).

A keyless signature blockchain infrastructure (KSBI) differs from proof or work blockchain based crypto currencies as it is based around a concept of permission- based blockchain - to provide widely witnessed evidence on what can be considered the truth, independently of any single party and while retaining complete confidentiality of the original data. Another unique characteristic that differentiates the KSBI from other distributed ledger solutions are its ability to scale to industrial applications to add one trillion data items to the blockchain every second, and to verify the data item from the blockchain within the next second. The ability to transact data at sub second speeds is essential to handle the increasing data requirements of a modern power grid (Mylrea et al. 2017).

KSBI is based on Guardtime’s patented technology keyless signature infrastructure (KSI_® which has been in production use since 2007, is employed by various world’s governments – i.e. Estonia and Defense primes in United States - and is beginning to see adoption in the private sector for application for their systems and networks. A KSBI may also help realize several cybersecurity and compliance goals for the energy sector, such as: 

Smart contracts: Smart contracts execute and record transaction in the blockchain load ledger through blockchain enabled advanced metering infrastructure (AMI). Blockchain based smart contracts may help facilitate consumer level exchange of excess generation from DER. This could provide additional storage and help substation load balancing from bulk energy systems. Moreover, smart contract data is secured in part through decentralized storage of all transactions of energy flows and business activities (Mylrea et al. 2017). 

Secure Data Storage in Cryptographically Signed Distributed Ledger: Blockchain can help fill various optimization and security gaps and improve the state of the art in grid resilience by providing an atomically verifiable cryptographic signed distributed ledger to increase the trustworthiness, integrity and security of energy delivery systems at the edge. Blockchain can be used to verify time, user, transaction data and protect this data with an immutable crypto signed distributed ledger (Mylrea et al. 2017).

Certainly, AI enabled blockchain will be disruptive and replace jobs, especially traditional 3rd parties that are replaced by new consensus algorithms and distributed trust mechanisms. Energy aggregators and meter readers could potentially be replaced by a dynamic distributed ledger. Blockchain innovation will also create new energy jobs, value, and markets. Even as technology empowers humans, it also changes the relationship between man and machine, technology and organizations, society and innovation. Autonomous blockchain organizations may distribute power and leadership via cryptographic votes that establish equity against a contract or even mission statement. For example, future energy organizations may have stakeholders govern what type of energy mix they would like and have that preference or willingness to pay be capture in a smart contract. Blockchain AI empowered energy organizations might be increasingly autonomous made up decentralized contractors and investors with power to vote, invest and delivery services based on an immutable smart contract that captures who, what, when and where services are executed and shared in a transparent immutable ledger.

The notion of a “self-bootstrapped” organizations with crypto equities leveraging independently contractors guided by decentralized blockchain voting has been explored (Levine 2014). Bit congress has established a blockchain based voting system. The country of Georgia is leveraging blockchain to facilitate real estate licensing. Estonia has established a privacy preserving secure virtual government using keyless signature infrastructure blockchain. These examples highlight how technology can help distribute trust and reduce redundancy in everything from billing to middle management, creating new value for organizations in an increasingly decentralized autonomous society. Reducing redundancy creates new value and more competitive organizations (Lawless 2017)


Blockchain and AI integration and innovation may present a more resilient and efficient path for decentralized cyber and physical devices to interactive, transforming modern infrastructure into array of smart autonomous systems of systems. Increased autonomy and control is essential to optimized the rapidly growing “Internet of Things” environment that Gartner has predicted to include 26 billion devices by 2020 (Gartner 2013).

“Simple and easy to write contracts appear to be sufficient for many entirely digital transactions. But as these systems start to interact with the physical world, there is likely to be a need for greater intelligence and real world knowledge in making decisions. AI systems will be needed to translate information from a wide variety of sensors into precise terms that smart contracts can act upon. In the other direction, contracts that lead to physical actions (such as delivery of items) will need to interface with human and robotic agents. For example, owner and operators of critical energy infrastructure might want insurance contracts against cyber-attacks and harmful weather conditions and a smart contract would need to determine when the payout event is triggered (Levine 2014).

These grid optimization, automation and resilience improvements are essential operations and design criteria as we modernize our power grid. However, cybersecurity is often an afterthought as vendors and end users prioritize functionality and cost, leaving our power grid, the backbone of our economy, potentially vulnerable to a cyber-attack. This is especially true at the grid’s edge which continues to increase the size and speed of data being collected and exchanged in absence of clear cybersecurity and IoT standards and regulation. Thus, the grid lacks the necessary defenses to prevent disruption and manipulation of DERs, grid edge devices and associated electricity infrastructure. Moreover, as the smart grid increases its connectivity and communications with buildings, cyber vulnerabilities will extend behind the meter into “smart” buildings, which also have a host of documented cybersecurity vulnerabilities.

Blockchain technology can also be applied to the smart grid to help reduce costs by cutting out 3rd parties and increasing the arbitrage opportunity for individuals to produce and sell energy to each other. Smart contracts facilitate peer-to-peer energy exchanges by enabling energy consumers and procures to sell to each other, instead of transacting through a multi-tiered system, in which distribution and transmission system operators, power producers, and suppliers transact on various levels (Mylrea and Gourisetti 2017). In April 2016, one of the first use cases was demonstrated where energy generated in a decentralized fashion was sold directly between neighbors in New York via a blockchain system, demonstrating that energy producers and energy consumers could execute energy supply contracts without involving a third-party intermediary; effectively increasing speed and reducing costs of the transaction (PWC 2017).

In addition to potential cost savings, transaction data might be more secure through decentralized storage and multifactor verification of transactions in the blockchain distributed ledger (PWC 2017). Blockchain reduces the need for 3rd parties to process transactions: Electricity is generated  Consumer buys the electricity  blockchain based meters update the blockchain, creating a unique timestamped block for verification in a distributed ledger: 1) At the distribution level, system operators can leverage the blockchain to receive energy transaction data to charge their network costs to consumers; 2) Reduces data requirements and increases speed of clearing transactions for transmission system operators as transactions could be executed and settled on the basis of actual consumption (Mylrea and Gourisetti 2017).

Smart contracts execute and record transaction in the blockchain load ledger through blockchain enabled advanced metering infrastructure (AMI). Blockchain based smart contracts can facilitate consumer level exchange of excess generation from DERs, EVs, etc. This could provide additional storage and help substation load balancing from bulk energy systems. Moreover, smart contract data is secured in part through decentralized storage of all transactions of energy flows and business activities. This highlights the disruptive potential for blockchain on energy markets through the introduction of a more autonomous and decentralized transaction model.

This peer to peer system may reduce or even replace the need for a meter operator if the meter blockchain is shared with the distribution system operator.

Currently, the power grid lacks the necessary security and resilience to prevent cyber-attacks on DERs, grid edge devices and associated electricity infrastructure. Cyber vulnerabilities and interoperability challenges also extend behind the meter into building automation and controls systems. Applying blockchain could help increase fidelity and security of buildings to grid communications. Moreover, multiple customers can leverage the same widely witnessed blockchain to cryptographically verify the other entities data when needed, creating a distributed trust mechanism. Blockchain may also help solve several optimization and reliability challenges that have been ushered in with grid modernization.

Currently, time-lags for payment and uncollected bills leaves value on the table and the real cost associated with the energy value chain is not captured. Blockchain can record real time net loads and smart contracts execute customers distributed generated sales and purchases. Currently, grid operators lack visibility and control of real- time power flows and injections from DERs and distributed generation customers. Blockchain can help optimize network data and record residual energy at the substation level. Increasing the fidelity and control of utility data will also help settle with bulk systems as well as negotiate future contracts.


Blockchain, AI and IoT have a lot of buzz right now. Reading the news one might assume that blockchain is a panacea for all that ills us – climate change, cyber security, volatile financial systems. AI articles suggest that robots are coming and may take our jobs. Internet of Things or IoT cyber incidents remind us that everything is increasingly connected to the internet and collecting and exchanging data that is potentially vulnerable. While these are disruptive in their own way and create some exciting new opportunities, many challenges remain. Several fundamental policy, regulatory and scientific challenges remain before blockchain realizes its disruptive potential. This sections explores some of the challenges as they relate to block chain’s application to the array of things.

Applying AI Blockchain to modernize electricity infrastructure also requires speed, agility and affordable technology. AI enhanced algorithms are not always cheap and often require prodigious data sets that must be broken down into a code that makes sense. However, there is a lot of noise or distracting data being exchanged in electricity infrastructure, making it difficult to identify what caused an anomaly – what is a software hire, cyber-attack, weather event, all the above? It can be very difficult to determine what normal looks like. Thus, developing an AI enhanced grid requires breaking down the data into observable patterns, which is also very challenging from a cyber perspective as threats are complex, non-linear and evolving.

New blockchain opportunities are also accompanied by the lack of policy, legal and regulatory frameworks. For example, even if some intermediaries are replaced in the energy sector, there still needs to be schedule and forecast submitted to the transmission system operator for electricity infrastructure to be reliable. Another challenge is incorporating individual blockchain consumers into a balancing group and having them comply with market reliability and requirements and submit accurate demand forecasts to the network operator. Managing a balancing group is not a trivial task and could potentially increase costs of managing the blockchain. To avoid costly disruptions, blockchain autonomous data exchanges, such as demand forecasts from the consumer to the network operator will need to be stress tested for security and reliability before deployed at scale.

Applying blockchain to modernizing and secure electricity infrastructure also presents several cyber security challenges. For example, Ethereum based smart contracts provide the ability for anyone to write electronic code that can be executed on a blockchain. For example, an energy producer or consumer agrees to buy or sell renewable energy from a neighbor for an agreed upon price that is captured in blockchain based smart contract. AI could help increase the efficiency and automate the auction to include other bidders and sellers in a more efficient and dynamic way, but this would require a lot more data and analysis of that data to recognize discernable patter in that data to inform the AI algorithm of the smart contract.

This also requires the code of the blockchain to be more resilient to cyber-attacks. Previously, Ethereum has shown to have several vulnerabilities that may underline the trustworthiness of this transaction mechanism. Vulnerabilities in the code have been exploited in at least three multi-million dollar cyber incidents. In June 2016, DAO, was hacked exploiting vulnerable smart contract code and extracting approximately $50 million dollars. In July 2017, vulnerable code in am Ethereum wallet was exploited to extract $30 million dollars of cryptocurrency. In January 2018, hackers stole roughly 58 billion yen ($532.6 million) from a Tokyo-based cryptocurrency exchange. Coincheck Inc. This incident highlighted the need for increased security and regulatory protection for cryptocurrencies and other blockchain applications. The Coincheck hack appears to have exploited vulnerabilities in a “hot wallet” which is a crypto currency wallet that is connected to the internet. In contrast, cold wallets, such as Trezor and Ledger Nano S, are cryptocurrency wallets that are stored offline.


Despite being a centralized currency, Coincheck was a centralized cryptocurrency exchange with a single point of failure. However, the blockchain shared ledger of the account may potentially be able to tag and follow the stolen coins and identify any account which receives them (Fadilpašić and Garlick 2018). Storing prodigious data sets that constantly growing on a blockchain can also create potential latency or bloat in the chain, requiring large amounts of ram and memory on a server. These requirements for ethereum based smart contracts have grown over time and the block takes a longer time to get processed. For time, sensitive energy transactions this may create speed, scale and cost issues of the smart contract is not designed properly.



#### 发明内容
__的创新设计是__

#### 具体实施方式

#### 总结

#### 附图说明



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
