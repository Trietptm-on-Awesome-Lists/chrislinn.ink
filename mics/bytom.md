# Bytom

## 我在比原的工作

+ bytom
    * 可以看看 release notes
    * PoW 相关完善及优化
        + 优化tensority
            * go 920ms -> go 820ms -> simd 160ms -> cuda 6ms
        + block recommit
    - consume newBlockCh for vault_mode
    - tx cache
        + orphan? blockcache?
        + chain
            * if nil, rescan
            * protocol/store.go
            * database/leveldb/...
        + api
            * add to getTx
            * wallet rescan?
            * force rescan?
        + wallet!
            * 临时关联输入输出?
            * a.rescanWallet()?
            * wallet/
                - TxPrefix
                - TxIndexPrefix
+ Precogs
    * 涉及 P2P
+ 中心化钱包
    * api & database schema
    * build tx
    * fee estimate
    * multisign
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
多重签名
多币种， stable coin
dapp

bycoin Main: management instead of payment
寄生 btc/eth

支付：
巴比特积分 btm
矿池打币

身份：
DID

Dapp

ico

 -->


<!-- 
# 2018_12_13 晚会议记录

## 这个版本
+ retire类型交易 utxo 是否忘了处理。应该变成不可用。
+ X. api 整合import和create，（即整合 list-guids 和 create），一个 pubkey 永远只返回特定guid。幂等性。如果是已存在的pubkey就返回之前数据库中存在的 guid
++ X. 即默认单账户体系，通过 传参"wallet_idx"来支持多账户体系
++ X better keep list-guids, to support multiple account.
++ opt query utxo
++ utxo >21 build fail
+++ availabel amount
+ submit tx 压测
+ list address 直接计算好并返回资产价值（现在只有数量），否则 如果多个资产，那么app就要请求完一次 list-address 后又要分资产多次请求 /q/asset 并计算
+ 路由改一改，account 可以全部改成 merchant
+ rename market
+ struct keeper
+ del expired unconfirmed txs

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
# TODOS
+ blockcenter
    * chainkd_util
    * time related
        - lock_until
        - submission
+ mining pool
    * research
        + MinDiff
        + DiffType
        + GetTargetHex
        + leading zero
        + retarget
        + processShare
            + s.hashrateExpiration, 
            + s.minuteHashrateExpiration
        + shareTimeRing
+ tensor
    * arm ver
    * asm ver
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
+ benchmark
+ AIHash struct
    * cache
    * Hash()
+ head_hash 和 锚点 是否可以优化？
+ 有没有 生成 doc 的文件
+ mining/tensority
    * __blockHeader & seed 怎么来的__
+ 搞清楚 BigEdian LittleEdian 的区别好吧
+ 看 asset 数据结构，搞清楚商业逻辑

# DONE
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
+ 3-12 Beijing meeting
    * 钱包-生成地址
    * 钱包-转帐（收钱？）
    * API 创建地址给矿工打钱
    * will `get work` 每个数据字节长度 change?
    * 网页调用 -> API调用
    * 单芯片  1s 验证100次
    * need coinbase addr
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
- mining addr
    + miningAddressKey
+ pingpong
    ```
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/icmp/ping_test.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/transport/transport_test.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/transport/http2_server.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/transport/http2_client.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/transport/control.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/transport/bdp_estimator.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/test/end2end_test.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/stress/client/main.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/interop/http2/negative_http2_client.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/interop/client/client.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/interop/test_utils.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/benchmark/latency/latency.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/google.golang.org/grpc/call_test.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/gonum.org/v1/gonum/lapack/gonum/dlasq2.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/websocket/websocket_test.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/websocket/websocket.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/websocket/hybi_test.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/websocket/hybi.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/publicsuffix/table_test.go:

    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/nettest/conntest_go17.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/nettest/conntest_go16.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/nettest/conntest.go:




    /home/gavin/work/go/src/github.com/bytom/vendor/gonum.org/v1/gonum/lapack/internal/testdata/dlasqtest/dlasq2.f:
    /home/gavin/work/go/src/github.com/bytom/vendor/gonum.org/v1/gonum/lapack/internal/testdata/dlasqtest/dlasq3.f:
    /home/gavin/work/go/src/github.com/bytom/vendor/gonum.org/v1/gonum/lapack/internal/testdata/dlasqtest/dlasq5.f:
    /home/gavin/work/go/src/github.com/bytom/vendor/gonum.org/v1/gonum/lapack/internal/testdata/dlasqtest/dlasq6.f:





    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/icmp/ping_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/icmp/example_test.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/h2i/h2i.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/h2i/README.md:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/h2demo/h2demo.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/writesched_random.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/write.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/transport_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/transport.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/server_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/server.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/not_go17.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/go17.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/frame_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/http2/frame.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/html/atom/table_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/html/atom/table.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/net/html/atom/gen.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/golang.org/x/crypto/ssh/handshake.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/wire/protocol.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/wire/msgpong.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/wire/msgping_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/wire/msgping.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/wire/message_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/wire/message.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/wire/doc.go:


    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/rpcclient/net.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/peer/peer_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/peer/peer.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/peer/log.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/peer/doc.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/peer/README.md:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/docs/json_rpc_api.md:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/btcjson/chainsvrresults.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/btcjson/chainsvrcmds_test.go:
    /home/gavin/work/go/src/github.com/bytom/vendor/github.com/btcsuite/btcd/btcjson/chainsvrcmds.go:
    ```
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


## 回答老马
* 为何比原链适合上链资产，比起以太坊、neo、bts有什么好处？
    * 以太坊
        * 比特币本身的中的非图灵完备栈式脚本语言，所表达的功能极少，很难实现一些稍微复杂的功能
        * 以太坊想设计一个功能强大完备的平台，但是太理想主义的结果就是，他们想做出很强大的东西但是没能做出来或者说没能做出效果好的来，性能不好。之前一个小养猫游戏就搞得以太坊奔溃。（从技术角度来讲，以太坊账户模型是面向对象的，为了易于管理账户，而引入了全局状态，每一笔交易都会改变这个全局状态。这和现实世界是相对应的，每一个改变，都会改变这个世界。这种设计很强大但是很难扩展，也无法并行运行，注定了他不能快速执行。）
        * 比原链的功能是介于比特币和以太坊之间。实现数字资产方面的图灵完备的智能合约即可。比比特币非图灵完备智能合约进半步，比以太坊包容万象的智能合约退半步。我们包含他们两者的优点（使用比特币的 UTXO 模型，加 轻量级虚拟机实现数字资产合约)。既有了比特币中UTXO的并行性（这样可以快速执行），又有了数字资产的发行和管理的功能。
    * neo
        - 今天段总发了一个链接，国外网友读了 neo 的代码发现 neo 不是去中心化的，信任节点是被写死的，那么一旦信任节点被黑，系统就不可信了
            + 但是我也还没查证，还是先查证比较好，造谣这个锅可背不起
    * bts
        - 采用dpos，类似董事会投票，缺点是虽然恶意的节点将在下一轮投票被踢出，但单个恶意区块在短期仍有可能是有效的状态。也就是说，不能保证链每时每刻都是正确可信的。
* 如果银行等合作方认为联盟链更适合其应用，我该如何向其说明？ 
    - 联盟链的算法缺陷
        + 复杂度太高，其广播时间随网络中节点数平方级增长，目前当系统中超过100个节点就会崩溃
        + 黑客攻击联盟链改写数据只需要攻击整个网络中 1/3 的节点，而攻击比原链需要攻击整个网络超过一半的节点
* 我理解，比原链的载体是一套程序？一套程序承载的去中心化协议？
    - 可以这么理解，准确的说，需要 程序+硬件(电脑)+用户 的配合
* 巴比特网站文章版权的上链是否探讨过？
    - 这个我不太清楚，但版权也属于资产对吧，我觉得是可行的。
* 上链资产如何达到形同“电子合同”般的效力。满足《电子签名法》精神，在生成（时间戳）、存储（我为第三方）、提取、身份认证，三个方面。
    - 不是学法律的，不太清楚。
* 资产上链的形式
    - 包含
        + 时间
        + 别名
        + 备注
        + 数量
        + 以及其他等等

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

### multi-sign
#### TODOS
+ [ ] multi-sign
    * [ ] addUTXO
    * [X] createWallet
    * [X] restoreWallet
+ [ ] delete unconfirmed utxo
+ [ ] 调研if txProposalSign.Signatures == "" {是否可以去除
+ [X] join multi-sign 进行派生
+ [X] txproposal memo
+ [X] newAddress handle mutilsign
+ [X] NewUtxoSelector 历史记录缺陷现在选择 utxo 只会一次选择在多少个之内，而不是一共选择了多少个
+ [X] JoinMultiSignWallet对pubkey进行合法性验证, 不然等生成地址的时候再判断就坑了
+ [X] db tx_proposals unique key
+ [X] txproposal lock untxo more than 24 hours， 7 days?
