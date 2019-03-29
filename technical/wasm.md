# WASM 关注列表

---

__阅读列表__

+ https://webassembly.eu/
+ https://wasmweekly.news/
+ https://github.com/webassembly
* https://rust.cc/
+ https://www.8btc.com/author/3708
* 玄武
* https://bbs.chainon.io/

---

+ https://webassembly.eu/
+ https://wasmweekly.news/
+ https://blogs.autodesk.com/autocad/autocad-web-app-google-io-2018/
+ https://zhuanlan.zhihu.com/p/36130095
+ https://zhuanlan.zhihu.com/p/42274816
+ http://webassembly.github.io/spec/
+ https://github.com/WebAssembly/design
+ https://webassembly.org/
+ https://www.w3.org/community/webassembly/
+ https://www.w3.org/wasm/
+ https://developer.mozilla.org/en-US/docs/WebAssembly
+ https://webassembly.org/docs/high-level-goals/
+ https://tomassetti.me/introduction-to-webassembly/
+ https://www.parity.io/wasm-smart-contract-development/
+ https://github.com/paritytech/parity-ethereum
+ https://github.com/rustwasm/team
    * https://github.com/rustwasm/wasm-pack
    * https://github.com/rustwasm/create-wasm-app
+ https://rustwasm.github.io/book/
+ https://github.com/rustwasm/awesome-rust-and-webassembly
+ https://developer.mozilla.org/zh-CN/docs/WebAssembly/Rust_to_wasm
+ https://github.com/mbasso/awesome-wasm
+ https://juejin.im/entry/5be3d6a2518825170678f0e6
+ https://juejin.im/entry/5b24d1406fb9a00e833d446e
+ https://juejin.im/entry/5bb476eff265da0aba70cb7f
+ https://juejin.im/entry/5b7688b2e51d455f997fc6a5
+ https://juejin.im/entry/5b7a3fcc518825430d26ae48
+ https://github.com/patterns/hello-wasm
+ https://github.com/patterns/wasm-websocket
+ https://github.com/patterns/wasm-life
+ JIT vs wasm
    * https://mp.weixin.qq.com/s/8KBiHp2rKckwJIZLpgh3eQ
+ 用 Webpack 运行 Rust 转 wasm
    * https://magicly.me/fe-hpc/rust-asmjs-and-webassembly/
    * https://imweb.io/topic/5c06b8b0611a25cc7bf1d7d5



__rust...__
适合区块链: 代码尺寸小，安全性高，可靠性高，资源使用最少

+ https://m.aliyun.com/yunqi/articles/62505
+ https://www.zhihu.com/question/31038569
+ https://www.zhihu.com/question/30407715
+ https://github.com/ctjhoa/rust-learning
+ https://research.mozilla.org/rust/
+ https://www.ibm.com/developerworks/cn/opensource/os-developers-know-rust/
+ https://github.com/RustStudy
    * __https://rust.cc/__
    * https://github.com/RustStudy/tao_of_rust_docs
+ https://doc.rust-lang.org/
    * https://rust-lang.org/
    * https://doc.rust-lang.org/stable/rust-by-example/
+ https://github.com/rust-lang
+ https://prev.rust-lang.org/en-US/documentation.html
+ https://www.rust-lang.org/learn
+ https://github.com/rust-lang-cn
    * https://rustlang-cn.org/office/rust/book/
        - poor translation
+ https://rustlang-cn.org/office/rust/
+ https://github.com/rust-unofficial/awesome-rust
+ https://github.com/not-yet-awesome-rust/not-yet-awesome-rust

__rust NG!__
+ https://github.com/rust-lang-nursery
    * https://rust-lang-nursery.github.io/rust-cookbook/


__iohk?__
+ https://cardanodocs.com/technical/plutus/introduction/
    * https://github.com/input-output-hk/plutus
    * https://github.com/input-output-hk/rust-cardano
    * https://github.com/input-output-hk/cardano-chain
    * https://github.com/input-output-hk/ouroboros-network
    * https://testnet.iohkdev.io/plutus/


## asm.js


## wasm代码的运行一共有三种模式

+ binaryen模式
    * 基于 bytecode
+ wavm模式
    * 基于JIT，速度可以达到运行native code的级别
    * cons
        - 致命硬伤：JIT时编译速度太慢
            + 以编译源代码里的eosio.system这个系统智能合约为例，需要5秒左右的时间。这已经不是慢的问题的，而是在每个块0.5秒的情况下，已经根本无法满足需求了。
        - 占用的内存比较大
    * pros
        - eos
            + 在replay的时候加快replay的速度。但是这会占用大量额外的内存。
            + 给特权级别的合约进行加速。最常用的就是eosio.token和eosio.system这两个合约。
            + 虽然不能被大量使用，但是可以把它定义为一种稀缺资源。像短帐户名称一样通过竞价的方式来获取。另一种更为简单的的方式就是就是让智能合约的账户来承担wavm JIT模式下的额外内存费用。
            + wavm JIT加速模式的加载时间比较长，所以必须采用异步加载的模式。在加载过程中，合约可以运行在wabt这种解释器的模式之下。
                * 但还要解决一个问题。就是wasm的代码目前只支持单线程运行，另外，由于wasm的设计问题，并不支持wavm和wabt两种模式的共存。也就是说，在程序的初始化的过程中必须指定一种运行模式，wabt模式或者wavm模式。但是在程序跑起来后，就只能以wabt模式或者wavm模式运行了，否则会因为内存冲突导致程序的崩溃。这个问题也应该是后面Eos的要面对的一个问题。PyEos的解决办法是采用将相同的代码编译成两个动态库，这两个动态库是不会共享内存的，从而解决了wavm模式和wabt模式下内存冲突导致程序崩溃的问题。
+ wabt模式

### .wabt
基于栈的bytecode模式, “汇编语言”版本, 采用“S-表达式（S-Expressions）”文本表示, 可以直接通过工具将 *.wat 文件编译为 *.wasm 文件

### .wasm
编译好的二进制文件

## WABT: The WebAssembly Binary Toolkit
+ wasm2wast tool
+ wast2wasm tool
+ wasm-interp tool
    * This is an interpreter that lets developers run a WebAssembly binary from the command line __stand-alone__. It implements a stack-machine based interpreter that interprets the WebAssembly binary directly. This differs from how __a browser would JIT__ the WebAssembly binary into native code for its target architecture at load time.


## Binaryen
outputs WebAssembly for compilers

It has a C API and implements its own internal intermediate representation (IR) of program logic and can perform a number of optimizations on the IR, support parallelization of code generation, etc.

For example, binaryen is used as part of the compiler asm2wasm that can convert asm.js files into WebAssembly files. It’s also used for supporting the LLVM compiler infrastructure generation of WebAssembly and compilation from Rust.