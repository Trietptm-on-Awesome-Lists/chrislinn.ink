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

1. 游戏框架参考 [gowog](https://github.com/giongto35/gowog)

2. 前端调用 scatter

```
import ScatterJS from 'scatterjs-core';
import ScatterEOS from 'scatterjs-plugin-eosjs';
import Eos from 'eosjs';

// Don't forget to tell ScatterJS which plugins you are using.
ScatterJS.plugins( new ScatterEOS() );

// Networks are used to reference certain blockchains.
// They let you get accounts and help you build signature providers.
const network = {
    blockchain:'eos',
    protocol:'https',
    host:'nodes.get-scatter.com',
    port:443,
    chainId:'aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906'
}

// First we need to connect to the user's Scatter.
ScatterJS.scatter.connect('My-App').then(connected => {

    // If the user does not have Scatter or it is Locked or Closed this will return false;
    if(!connected) return false;

    const scatter = ScatterJS.scatter;

    // Now we need to get an identity from the user.
    // We're also going to require an account that is connected to the network we're using.
    const requiredFields = { accounts:[network] };
    scatter.getIdentity(requiredFields).then(() => {

        // Always use the accounts you got back from Scatter. Never hardcode them even if you are prompting
        // the user for their account name beforehand. They could still give you a different account.
        const account = scatter.identity.accounts.find(x => x.blockchain === 'eos');

        // You can pass in any additional options you want into the eosjs reference.
        const eosOptions = { expireInSeconds:60 };

        // Get a proxy reference to eosjs which you can use to sign transactions with a user's Scatter.
        const eos = scatter.eos(network, Eos, eosOptions);

        // ----------------------------
        // Now that we have an identity,
        // an EOSIO account, and a reference
        // to an eosjs object we can send a transaction.
        // ----------------------------


        // Never assume the account's permission/authority. Always take it from the returned account.
        const transactionOptions = { authorization:[`${account.name}@${account.authority}`] };

        eos.transfer(account.name, 'helloworld', '1.0000 EOS', 'memo', transactionOptions).then(trx => {
            // That's it!
            console.log(`Transaction ID: ${trx.transaction_id}`);
        }).catch(error => {
            console.error(error);
        });

    }).catch(error => {
        // The user rejected this request, or doesn't have the appropriate requirements.
        console.error(error);
    });
});
```

3. 游戏结束后，发起解锁



## BTM
### bet
```
contract BetGame(status: Hash, hashedAnswer: Hash, totalEven: Integer, totalOdd: Integer) locks valueAmount of valueAsset {
  clause bid(amount: Integer, bidSide: Integer, bidder: Program) {
    define temp1:String = concat(status, amount)
    define temp2:String = concat(temp1, bidSide)
    define result:String = concat(temp2, bidder)
    define newStatus:Hash = sha3(result)
    if bidSide%2 == 1 {
       lock valueAmount + amount of valueAsset with BetGame(newStatus, hashedAnswer, totalEven+amount, totalOdd)
    } else {
        lock valueAmount + amount of valueAsset with BetGame(newStatus, hashedAnswer, totalEven, totalOdd+amount)
    }
  }
  clause check(answer: Integer, preStatus: Hash, amount: Integer, bidSide: Integer, bidder: Program) {
    verify sha3(answer) == hashedAnswer
    define temp1:String = concat(preStatus, amount)
    define temp2:String = concat(temp1, bidSide)
    define result:String = concat(temp2, bidder)
    verify sha3(result) == status
    if (bidSide%2) == (answer%2) {
       define total:Integer = totalOdd
       if (bidSide%2) == 1 {
         assign total = totalEven
       }
       define reward:Integer = (totalEven+totalOdd)*amount/total
       lock reward of valueAsset with bidder
       lock valueAmount - reward of valueAsset with BetGame(preStatus, hashedAnswer, totalEven, totalOdd)
    } else {
       lock valueAmount of valueAsset with BetGame(preStatus, hashedAnswer, totalEven, totalOdd)
    }
  }
}
```

### price changer
```
contract PriceChanger(askAmount: Amount, askAsset: Asset, sellerKey: PublicKey, sellerProg: Program) locks valueAmount of valueAsset {
    clause changePrice(newAmount: Amount, newAsset: Asset, sig: Signature) {
      verify checkTxSig(sellerKey, sig)
      lock valueAmount of valueAsset with PriceChanger(newAmount, newAsset, sellerKey, sellerProg)
    }
    clause redeem() {
      lock askAmount of askAsset with sellerProg
      unlock valueAmount of valueAsset
    }
  }
```