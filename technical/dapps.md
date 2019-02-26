# DApps

最近在开发半去中心化游戏，记录一下自己的一些想法。

为了游戏体验，先支持 pos，面向 eos trx，先支持 scatter。

拟支持游戏列表

+ 单人吃豆人
+ 黄金矿工
+ 多人吃豆人
+ 多人贪吃蛇
+ 五子棋
+ 井字棋


后续支持:

+ 手机版
+ 手机绑定? 微信登陆？
+ 先游戏再支付

## ETH EOS TRX

### 项目架构

1. 游戏框架参考
    + [gowog](https://github.com/giongto35/gowog)
        * 并发好，支持位置。但是 用的是 protobuf
    + [BrowserQuest](https://github.com/mozilla/BrowserQuest)
        * socket.io
        - 多星
        - 玩家系统
        - 没跑起来
            * [BrowserQuest-PHP](https://github.com/walkor/BrowserQuest-PHP) 跑起来了
    + [SquareOff](https://github.com/ScriptaGames/SquareOff/)
        * 还行
    + [Space-Shooter](https://github.com/Couchfriends/Space-Shooter)
        * 用户系统 基于 Couchfriends，放弃
    * [breakout](https://github.com/Couchfriends/breakout)
        * 用户系统 基于 Couchfriends，放弃
    * [I-Spy-A-Ghost](https://github.com/OmarShehata/I-Spy-A-Ghost)
    * [jumpsuit](https://github.com/KordonBleu/jumpsuit)
    * [blk-game](https://github.com/morozd/blk-game)
        * 编译没成功
2. 前端调用 scatter

3. 游戏结束后，发起解锁
