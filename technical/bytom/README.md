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

# Bytom Test
## validate_ _tx

`func ValidateTx(tx *bc.Tx, block *bc.Block) (*GasState, error)` 对交易进行验证

交易除了原生 types 层之外，有个 map 层的封装，便于 验证

```
type validationState struct {
    block     *bc.Block
    tx        *bc.Tx
    gasStatus *GasState
    entryID   bc.Hash           // The ID of the nearest enclosing entry
    sourcePos uint64            // The source position, for validate ValueSources
    destPos   uint64            // The destination position, for validate ValueDestinations
    cache     map[bc.Hash]error // Memoized per-entry validation results
}
```

`gasStatus := &GasState{GasValid: false}` 一开始是 false，然后随着验证逐步更新， 如果最终有效  GasValid  才为 true
