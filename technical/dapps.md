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
    + [boardgame.io](https://github.com/nicolodavis/boardgame.io)
    + [gowog](https://github.com/giongto35/gowog)
        * 并发好，支持位置。但是 用的是 protobuf
    + [pomelo](https://github.com/NetEase/pomelo)'s [treasures](https://github.com/NetEase/treasures) & []()
        * issue
            - https://github.com/fantasyni
            - https://github.com/laispace
            - https://github.com/NetEase/pomelo/issues/1121
                + https://github.com/node-pinus/pinus
                + https://github.com/topfreegames/pitaya
                + https://github.com/lonng/nano
                    * https://github.com/lonnng/nanoserver
                        - ref for desk
        * .
            ```
            幸好跑去看了一下 别的 repo
            不过 lordofpomelo 还是跑不起来，好在对我们意义不大。
            在 issue 里面看到了不少讨论，说网易已经不维护这个项目了，网易的人出来说很快会开源一个更好的，又有人吐槽 说前几年就这么说了

            看到帖子下面有人推荐替代品，
            https://github.com/node-pinus/pinus（这个还算活跃程度还可以但其实就是作者在自娱自乐，而且就是纯粹把pomelo做成ts版，而且暂时还没把 treasure 和 lordofpomelo 移植过来 ）
            https://github.com/topfreegames/pitaya（golang后端， 无前端）
            https://github.com/lonng/nano（golang ，并开源了个 带业务逻辑的后端 https://github.com/lonng/nanoserver）

            我目前能跑起来的就是  treasure 和 gowog 这两个 
            都带前后端，还是很好上手的，
            目前的打算是   先参考 nanoserver 着 设计一下room/desk&业务逻辑
            ```
    * [这里面的](https://github.com/leereilly/games) Browser-Based multiplayer game
        * [BrowserQuest](https://github.com/mozilla/BrowserQuest)
            - socket.io
            + 多星
            + 玩家系统
            + 没跑起来
                - [BrowserQuest-PHP](https://github.com/walkor/BrowserQuest-PHP) 跑起来了
        * [SquareOff](https://github.com/ScriptaGames/SquareOff/)
            - 还行
        - [jumpsuit](https://github.com/KordonBleu/jumpsuit)
            - 适合 建房间，不过 每开一个房间都要建一个server
            - 因为 ECMAScript Modules 和 import 语法问题还没有搭建成功
        * [Space-Shooter](https://github.com/Couchfriends/Space-Shooter)
            - 玩家系统 基于 Couchfriends，放弃
        - [breakout](https://github.com/Couchfriends/breakout)
            - 玩家系统 基于 Couchfriends，放弃
        - [I-Spy-A-Ghost](https://github.com/OmarShehata/I-Spy-A-Ghost)
            + 玩家系统使用 peerjs & PeerCloud，较麻烦
        - [blk-game](https://github.com/morozd/blk-game)
            - 编译没成功
2. 前端调用 scatter

3. 游戏结束后，发起解锁
