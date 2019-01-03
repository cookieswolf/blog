# MAC下搭建以太坊私有链

## 安装客户端

安装Gth，即Go语言实现的以太坊客户端 （go-ethereum)。

```
brew tap ethereum/ethereum
brew install ethereum
```

## 配置初始状态

配置私链网络的初始状态，新建 genesist.json

```
{  
    "config": {  
          "chainId": 22,  
          "homesteadBlock": 0,  
          "eip155Block": 0,  
          "eip158Block": 0  
      },  
    "alloc"      : {},  
    "coinbase"   : "0x0000000000000000000000000000000000000000",  
    "difficulty" : "0x20000",  
    "extraData"  : "",  
    "gasLimit"   : "0x2fefd8",  
    "nonce"      : "0x0000000000000042",  
    "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",  
    "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",  
    "timestamp"  : "0x00"  
}
```

chainId 制定了独立的区块链网络的ID，不同网络的节点无法互相连接。配置文件还对当前挖矿难度 difficulty，区块gas消耗限制 gasLimit进行了配置。

## 初始化区块链

```
geth --datadir "~/ethdev" init ethdev/genesis.json
```

## 以太坊客户端启动：

```
geth --identity "TestNode" --rpc --rpcport "8545" --datadir ethdev/ --port "30303" --nodiscover console 
```

登录成功以后可以查看帐户信息，当前有哪些帐户

```
> eth.accounts

["0xd15463b5ca866e1102b7bcb7ea72dda4203dbc74"]
```

## 创建账户

我本地创建已经创建了两个帐户

再创建一个：

```
> personal.newAccount('123')
"0x49d2b94b9da2b7f224b5b6b00aa77692dcad31fc"
```

以上通过personal.newAccount 传入密码参数即可创建一个新帐户并返回新的帐户地址；

在keystore里可以看到有新建好的账户文件

![](https://upload-images.jianshu.io/upload_images/4834364-02a779456d325285.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/826/format/webp)

这个文件要保存好，不要泄露给其他人。

在以太坊客户端 可以把某个帐户赋值给一个变量

```
 > user1=eth.accounts[0]
"0xd15463b5ca866e1102b7bcb7ea72dda4203dbc74"
```

然后可以查看当前帐户的余额:

```
> eth.getBalance(user1)
0
```

```
> eth.blockNumber
0
```

可以查看当前user1的余额为0，默认也是0个区块，因为还没有启动矿挖矿

## 启动挖矿

```
> miner.start()
true
```

在上面打开发文件监控界面就会看到 挖矿建块儿情况：

再 切回以太坊挖制台，查看用户余额:

```
> eth.getBalance(user1)

1.25890625e+21
```

```
> eth.getBalance(user2)
0
```

已经看到 帐户1，已经有余额了，帐户2 还是0，因为挖矿的奖励进入第一个帐户中。

现在停止挖矿：

```
> miner.stop()
true
```

已经停止成功，另外日志界面也停止输出

再 查看一下当前的区块高度：

```
> eth.blockNumber

288
```

现在帐户2中没有余额，我们从帐户1转发几个以太币到 帐户2中：

```
> eth.sendTransaction({from: user1,to: user2,value: web3.toWei(3,"ether")})

account is locked

    at web3.js:3119:20

    at web3.js:6023:15

    at web3.js:4995:36

    at <anonymous>:1:1
```

由于默认帐户是锁定的，首要解锁帐户，然后再 转帐,先查看 下下当前帐户，再解锁：

```
> eth.accounts

["0x73e8655a84a37685d98891b7a9333a7423e12cb3", "0xa9d6dfff13c1050f19a8ffc2811c68842797d01c", "0xe30cecc37776895389b94033ac65eb3b98294659"]
```

```
> personal.unlockAccount('0x73e8655a84a37685d98891b7a9333a7423e12cb3','11111111')

true
```

上面已经提示解锁成功，然后继续转帐：

```
> eth.sendTransaction({from: user1,to: user2,value: web3.toWei(3,"ether")})

"0x8f164a1296b618bdd64fcc007f6d39ce022b57e257beefeb76288cdef220ad80"

> eth.getBalance(user2)

0
```

上面已经提示转帐成功了，但是user2帐户余额依然是0，是因为没有矿工来挖矿处理，我们启动一个矿工，并在另一个终端查看日志

日志已经显示开始挖矿并发交易进行了处理在，区块293中，

```
> miner.start()

true
```

```
> miner.stop()

true
```

```
> eth.getBalance(user2)

3000000000000000000
```

我们再 次查看余额的时候user2已经 有三个以太币了，这样一个转帐的交易就完成 了；

参考文档:[搭建以太坊私链挖矿](https://www.jianshu.com/p/b154398014b0)










