# GoLang

## CSP 并发模型, communicating sequential processes
不同于传统的多线程通过共享内存来通信，CSP讲究的是以通信的方式来共享内存。

+ 传统
    * 线程间通信都是通过共享内存。
    * 非常典型的方式是锁，因此，衍生出一种方便操作的“线程安全的数据结构”。
    * go 也有
        - 临界区(critical section), 每次只允许一个goroutine进入某个代码块
            + 互斥锁
            + 条件变量 Cond
                * Wait
                * Signal
                * Broadcast 
        - 原子操作(atomicity)
            + Add
            + CAS, compare and swap 交换并比较
            + store value & load value
            + swap

### Don't communicate by sharing memory; share memory by communicating

+ [Share Memory By Communicating](https://blog.golang.org/share-memory-by-communicating)
+ [Explain: Don't communicate by sharing memory; share memory by communicating](https://stackoverflow.com/questions/36391421/explain-dont-communicate-by-sharing-memory-share-memory-by-communicating):
    + Instead of explicitly using locks to mediate access to shared data, Go encourages the use of channels to pass references to data between goroutines. 
    + an [interesting history](https://swtch.com/~rsc/thread/) that begins with C. A. R. Hoare's [Communicating Sequential Processes](http://www.usingcsp.com/).
+ [Codewalk: Share Memory By Communicating](https://golang.org/doc/codewalk/sharemem/):
    + [Channels](#channel) allow you to pass references to data structures between goroutines. If you consider this as passing around ownership of the data (the ability to read and write it), they become a powerful and expressive synchronization mechanism.
    + the convention is that sending a Resource pointer on a channel __passes ownership__ of the underlying data from the sender to the receiver. Because of this convention, we know that no two goroutines will access this Resource at the same time. This means we __don't have to worry about locking__ to prevent concurrent access to these data structures. 

### channel

## pprof

## [Effective Go](https://golang.org/doc/effective_go.html)


## Mem leak & GC
+ 减少对象分配
    * 尽量做到对象的重用
+ goroutine channel leak
    * 绝对不能由消费者关channel
        - 因为向关闭的channel写数据会panic。
        - 正确的姿势是生产者写完所有数据后，关闭channel，消费者负责消费完channel里面的全部数据
            ```
            func produce(ch chan<- T) {
                defer close(ch) // 生产者写完数据关闭channel
                ch <- T{}
            }
            func consume(ch <-chan T) {
                for _ = range ch { // 消费者用for-range读完里面所有数据
                }
            }
            ch := make(chan T)
            go produce(ch)
            consume(ch)
            ```
            + 为什么consume要读完channel里面所有数据？
                * 因为 `go produce()`可能有多个，这样可以确定所有 produce 的 goroutine 都退出了，不会泄漏。
    * 利用关闭channel来广播取消动作
        - 对于每个长连接请求各开了一个读取和写入协程，全部采用endless for loop不停地处理收发数据。当连接被远端关闭后，如果不对这两个协程做处理，他们依然会一直运行，并且占用的channel也不会被释放…这里就必须十 分注意，在不使用协程后一定要把他依赖的channel close并通过再协程中判断channel是否关闭以保证其退出。
+ 不要光盯着top上面的数字
    * 因为Go向系统申请的内存不使用后，也不会立刻归还给系统。
        - 只是告诉系统这些内存可以回收；操作系统并不是立即回收，等 到系统内存紧张时才会开始回收
+ gc stop the world, gc 过多会抢占程序的正常执行时间
+ go的 垃圾回收有个触发阈值，这个阈值会随着每次内存使用变大而逐渐增大（如初始阈值是10MB则下一次就是20MB，再下一次就成为了40MB…），如果长时 间没有触发gc go会主动触发一次（2min）。
+ 关注(从系统申请的内存会在Go的内存池管理，整块的内存页，长时间不被访问并满足一定条件后，才归还给操作系统。又因为有GC，堆内存也不能代表内存占用，清理过之后剩下的，才是实际使用的内存。)
    * 程序占用的系统内存
    * Go的堆内存
    * 实际使用到的内存
+ 使用
    * `runtime.ReadMemStats`: Go 内存使用信息
    * pprof
+ 考虑到程序中为了更好地做抽象，使用了反射操作，而 `reflect.Value` 会将对象拷贝并分配到堆上，程序中的对象都是消息体，有的消息体会超大，因此会分配较多的堆内存。对程序做了一版优化，去掉这个反射逻辑，改为 `switch case`
+ `fmt.Sprint`, 这个函数会把对象分配到堆上

## pitfalls
+ copy, zcc
+ sync/atomic/


## pool
+ https://golang.org/pkg/sync/#Pool
+ https://www.reddit.com/r/golang/comments/2ap67l/when_to_use_syncpool_and_when_not_to/
+ https://www.reddit.com/r/golang/comments/6ng0aq/correct_use_of_syncpool/
+ http://www.akshaydeo.com/blog/2017/12/23/How-did-I-improve-latency-by-700-percent-using-syncPool/
+ https://stackoverflow.com/questions/50851421/sync-pool-is-much-slower-than-using-channel-so-why-should-we-use-sync-pool


## bytom-cc
bytom coding code

### casting check
uint 转 int 要检查会不会 变成负数

尤其是 存在 用户输入的  p2p交换 和 合约


### avoid copying arrays in loops
https://github.com/ethereum/go-ethereum/pull/17265/

https://go-critic.github.io/overview.html#rangeexprcopy


### rangeValCopy
Avoid copying big objects during each iteration.

Use index access or take address and make use pointer instead.

https://go-critic.github.io/overview.html#rangevalcopy


### builtinshadow
变量名不要和关键字重复， 比如 `len`, `error`

https://go-critic.github.io/overview.html#builtinshadow

### submodule 
+ Delete the relevant section from the .gitmodules file.
+ Stage the .gitmodules changes git add .gitmodules
+ Delete the relevant section from .git/config.
+ Run git rm --cached path_to_submodule (no trailing slash).
+ Run rm -rf .git/modules/path_to_submodule (no trailing slash).
+ ~~Commit git commit -m "Removed submodule "~~
+ ~~Delete the now untracked submodule files rm -rf path_to_submodule~~

### map mutex


### append

https://stackoverflow.com/questions/27622083/performance-slices-of-structs-vs-slices-of-pointers-to-structs

AppendingStructs is faster than AppendingPointers