# DApps

最近在开发半去中心化游戏，记录一下自己的一些想法。

为了游戏体验，先支持 pos，面向 eos trx，先支持 scatter。

拟支持游戏列表

+ 单人吃豆人
+ 黄金矿工
    * X 延时不公平
+ 多人吃豆人
+ 多人贪吃蛇
+ 五子棋
+ 井字棋
+ sheji
+ eluosifangkuai
+ yinyou


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


# LAPP

+ code
    * https://github.com/bcongdon/awesome-lightning-network
    * https://github.com/ElementsProject/woocommerce-gateway-lightning
    * https://github.com/ElementsProject/wordpress-lightning-publisher
    * https://github.com/ElementsProject/filebazaar
    * https://github.com/ElementsProject/paypercall
    * https://github.com/ElementsProject/lightning-jukebox
    * https://github.com/ElementsProject/nanotip
    * https://github.com/ElementsProject/ifpaytt
+ nodes
    * https://1ml.com/
+ knowledge
    * https://segmentfault.com/a/1190000016454672
    * https://medium.com/cryptocow/lightning-vs-raiden-1-can-watchtowers-and-monitoring-services-scale-f3b59906114b
    * https://s1.rylink.com/info_detail/239
    * instamyna
        ```
        //to verify -> copy your values and set seedObject as below:
        var seedObject = {
          serverSeed: "1d26867264f68d6d1f503582dd6903ee", clientSeed: "42a00c71f192f75fca40fbf83b0cf8d3"};

        verifyServerSeedvsHash(seedObject);

        for(var i=1; i < 100; i++) {
          calcSeed(seedObject.serverSeed, seedObject.clientSeed, i);
        }


        function verifyServerSeedvsHash(seedObject) {
          var server_seed = seedObject.serverSeed;
            let hash = crypto.createHash('SHA256');
              hash.update(server_seed);
              var server_seed_hashed = hash.digest().toString('hex');
              console.log("server_seed: ", server_seed, "\nserver_seed_hashed: ", server_seed_hashed);
        }

        function calcSeed(server_seed, client_seed, nonce) {

              //var nonce = 1;
              var newclientseed = client_seed + "-" + nonce;
              //console.log("calcSeed newclientseed: ", newclientseed);

              let hmac = crypto.createHmac('SHA512', server_seed);
              hmac.update(newclientseed);
              let buf = hmac.digest();
              var finalOutput = buf.readUInt32BE() / Math.pow(2, 32) * 100;
              var winorlose = "lose";
              if(finalOutput < 45) {
                winorlose = "win";
              } 
              console.log("Bet Result: ",nonce, finalOutput, winorlose);
              return finalOutput;

        }
        ```
