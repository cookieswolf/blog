# 星云链Dapp开发(三):运行星云链

## 创世区块配置

在项目根目录下`conf/default/genesis.conf`,为创世区块配置文件

```
meta {
  # 每条链的唯一标识
  # 每个区块和交易只会属于一条唯一的链，保证安全性
  chain_id: 100
}

consensus {
  # 在贡献度证明(PoD)被充分验证前，星云链采用DPoS共识算法
  # DPoS共识中，21个人组成一个朝代
  # 每隔一段时间都会切换朝代，每个朝代内，21个矿工轮流出块
  # 由于DPoS只是过渡方案，所以暂时不开放给公众挖矿，即当前版本朝代不会发生变更
  dpos {
    # 初始朝代，包含21个初始矿工地址
    dynasty: [
      [ miner address ],
      ...
    ]
  }
}

# 预分配的代币
token_distribution [
  {
    address: [ allocation address ]
    value: [ amount of allocation tokens ]
  },
  ...
]
```

在此，我们使用默认配置即可，暂不用修改。

## 星云节点配置

在项目根目录下的`conf/default/config.conf`，为星云节点配置文件

```
# 网络配置
network {
  # 对于全网第一个节点，不需要配置seed
  # 否则，其他节点启动时需要配置seed，seed节点将会把网络中其他节点的路由信息同步给刚启动的节点
  # 可以配置多个seed, ["...", "..."]
  seed: ["/ip4/127.0.0.1/tcp/8680/ipfs/QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP"]

  # 节点监听网络消息端口，可以配置多个
  listen: ["0.0.0.0:8680"]

  # 网络私钥，用于确认身份节点
  # private_key: "conf/network/id_ed25519"
}

# 链配置
chain {
  # 链的唯一标识
  chain_id: 100

  # 数据存储地址
  datadir: "data.db"

  # 账户keystore文件存储地址
  keydir: "keydir"

  # 创世区块配置
  genesis: "conf/default/genesis.conf"

  # 签名算法，请勿修改
  signature_ciphers: ["ECC_SECP256K1"]

  # 矿工地址，矿工的keystore文件需要放置在配置的keydir下
  miner: "n1XkoVVjswb5Gek3rRufqjKNpwrDdsnQ7Hq"

  # Coinbase地址，该地址用于接收矿工的挖矿奖励，可以和矿工地址一致
  # 该地址的keystore无需暴露，不用放置在配置的keydir下
  coinbase: "n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"

  # 矿工地址的密码
  passphrase: "passphrase"
}

# API配置
rpc {
    # GRPC服务端口
    rpc_listen: ["127.0.0.1:8684"]

    # HTTP服务端口
    http_listen: ["127.0.0.1:8685"]

    # 开放的API模块
    # API模块包含所有和用户私钥无关的接口
    # Admin模块包含所有和用户私钥相关的接口，需要慎重考虑该模块的访问权限
    http_module: ["api", "admin"]
}

# 日志配置
app {
    # 日志级别: 支持[debug, info, warn, error, fatal]
    log_level: "info"

    # 日志存放位置
    log_file: "logs"

    # 是否打开crash report服务
    enable_crash_report: false
}

# 监控服务配置
stats {
    # 是否打开监控服务
    enable_metrics: false

    # 监控服务将数据上传到Influxdb
    # 配置Influxdb的访问信息
    influxdb: {
        host: "http://localhost:8086"
        db: "nebulas"
        user: "admin"
        password: "admin"
    }
}
```

在此，我们使用默认配置即可，暂不用修改。

## 矿工节点配置

在项目根目录下的`conf/example/miner.conf`，为矿工节点配置文件

```
network {
  # seed: "UNCOMMENT_AND_SET_SEED_NODE_ADDRESS"
  seed: ["/ip4/127.0.0.1/tcp/8680/ipfs/QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP"]
  listen: ["0.0.0.0:8780"]
  network_id: 1
}

chain {
  chain_id: 100
  datadir: "miner.1.db"
  keydir: "keydir"
  genesis: "conf/default/genesis.conf"
  start_mine: true
  # 总共21个矿工，在我们目前的测试环境中，由于我们只启动了21个矿工中的一个，就是coinbase，挖矿奖励将放入该账户
  coinbase: "n1XkoVVjswb5Gek3rRufqjKNpwrDdsnQ7Hq"
  miner: "n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"
  # 密码
  passphrase: "passphrase"
  # 加密算法
  signature_ciphers: ["ECC_SECP256K1"]
}

rpc {
    rpc_listen: ["127.0.0.1:8784"]
    http_listen: ["127.0.0.1:8785"]
    http_module: ["api","admin"]

    # http_cors: []
}

app {
    log_level: "debug"
    log_file: "logs/miner.1"
    enable_crash_report: true
}

stats {
    enable_metrics: false
    influxdb: {
        host: "http://localhost:8086"
        db: "nebulas"
        user: "admin"
        password: "admin"
    }
}
```

## 启动星云链

此时启动的星云链是本地的私有链，和官方的测试网和主网没有任何相互关联

- 第一步：启动你的第一个星云节点。

```
cd $GOPATH/src/github.com/nebulasio/go-nebulas
./neb -c conf/default/config.conf
```

如果看到如下信息说明节点启动成功

```
INFO[2018-10-31T11:33:28+08:00] Started crash reporter.                       file=crashclient.go func=main.InitCrashReporter line=115
INFO[2018-10-31T11:33:28+08:00] Setuping Neblet...                            file=neblet.go func="neblet.(*Neblet).Setup" line=115
INFO[2018-10-31T11:33:28+08:00] Starting pprof...                             file=asm_amd64.s func=runtime.goexit line=2362 listen="0.0.0.0:8888"
INFO[2018-10-31T11:33:28+08:00] Tail Block.                                   file=neblet.go func="neblet.(*Neblet).Setup" line=161 tail="{\"height\": 1, \"hash\": \"0000000000000000000000000000000000000000000000000000000000000000\", \"parent_hash\": \"0000000000000000000000000000000000000000000000000000000000000000\", \"acc_root\": \"db2a692aa8e21ba3a65fb952f441c5b346db29b3d4d10a7530b024e0ffc27050\", \"timestamp\": 0, \"tx\": 1, \"miner\": \"\", \"random\": \"\"}"
INFO[2018-10-31T11:33:28+08:00] Latest Irreversible Block.                    block="{\"height\": 1, \"hash\": \"0000000000000000000000000000000000000000000000000000000000000000\", \"parent_hash\": \"0000000000000000000000000000000000000000000000000000000000000000\", \"acc_root\": \"db2a692aa8e21ba3a65fb952f441c5b346db29b3d4d10a7530b024e0ffc27050\", \"timestamp\": 0, \"tx\": 1, \"miner\": \"\", \"random\": \"\"}" file=neblet.go func="neblet.(*Neblet).Setup" line=161
INFO[2018-10-31T11:33:28+08:00] Setuped Neblet.                               file=neblet.go func="neblet.(*Neblet).Setup" line=175
INFO[2018-10-31T11:33:28+08:00] Starting Neblet...                            file=neblet.go func="neblet.(*Neblet).Start" line=201
INFO[2018-10-31T11:33:28+08:00] Starting NebService...                        file=net_service.go func="net.(*NebService).Start" line=58
INFO[2018-10-31T11:33:28+08:00] Starting NebService Dispatcher...             file=dispatcher.go func="net.(*Dispatcher).Start" line=85
INFO[2018-10-31T11:33:28+08:00] Starting NebService Node...                   file=node.go 
```

## 第二步：启动你的第一个矿工节点，它的seed节点即我们刚刚启动的第一个节点。

```
cd $GOPATH/src/github.com/nebulasio/go-nebulas
./neb -c conf/example/miner.conf
```

在这个节点启动后，你会先看到如下信息，表示当前节点正在找种子节点同步。


```
INFO[2018-10-31T13:50:11+08:00] Setuping Neblet...                            file=neblet.go func="neblet.(*Neblet).Setup" line=115
INFO[2018-10-31T13:50:11+08:00] Genesis Configuration.                        consensus.dpos.dynasty="[n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE n1GmkKH6nBMw4rrjt16RrJ9WcgvKUtAZP1s n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so n1JAy4X6KKLCNiTd7MWMRsVBjgdVq5WCCpf n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS n1LmP9K8pFF33fgdgHZonFEMsqZinJ4EUqk n1MNXBKm6uJ5d76nJTdRvkPNVq85n6CnXAi n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz n1NwoSCDFwFL2981k6j9DPooigW33hjAgTa n1PfACnkcfJoNm1Pbuz55pQCwueW1BYs83m n1Q8mxXp4PtHaXtebhY12BnHEwu4mryEkXH n1RYagU8n3JSuV4R7q4Qs5gQJ3pEmrZd6cJ n1SAQy3ix1pZj8MPzNeVqpAmu1nCVqb5w8c n1SHufJdxt2vRWGKAxwPETYfEq3MCQXnEXE n1SSda41zGr9FKF5DJNE2ryY1ToNrndMauN n1TmQtaCn3PNpk4f4ycwrBxCZFSVKvwBtzc n1UM7z6MqnGyKEPvUpwrfxZpM1eB7UpzmLJ n1UnCsJZjQiKyQiPBr7qG27exqCLuWUf1d7 n1XkoVVjswb5Gek3rRufqjKNpwrDdsnQ7Hq n1cYKNHTeVW9v1NQRWuhZZn9ETbqAYozckh n1dYu2BXgV3xgUh8LhZu8QDDNr15tz4hVDv]" file=blockchain.go func="core.(*BlockChain).Setup" line=179 meta.chainid=100 token.distribution="[address:\"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE\" value:\"5000000000000000000000000\"  address:\"n1GmkKH6nBMw4rrjt16RrJ9WcgvKUtAZP1s\" value:\"5000000000000000000000000\"  address:\"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so\" value:\"5000000000000000000000000\"  address:\"n1JAy4X6KKLCNiTd7MWMRsVBjgdVq5WCCpf\" value:\"5000000000000000000000000\"  address:\"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS\" value:\"5000000000000000000000000\"  address:\"n1LmP9K8pFF33fgdgHZonFEMsqZinJ4EUqk\" value:\"5000000000000000000000000\"  address:\"n1MNXBKm6uJ5d76nJTdRvkPNVq85n6CnXAi\" value:\"5000000000000000000000000\"  address:\"n1NrMKTYESZRCwPFDLFKiKREzZKaN1nhQvz\" value:\"5000000000000000000000000\"  address:\"n1NwoSCDFwFL2981k6j9DPooigW33hjAgTa\" value:\"5000000000000000000000000\"  address:\"n1PfACnkcfJoNm1Pbuz55pQCwueW1BYs83m\" value:\"5000000000000000000000000\"  address:\"n1Q8mxXp4PtHaXtebhY12BnHEwu4mryEkXH\" value:\"5000000000000000000000000\"  address:\"n1RYagU8n3JSuV4R7q4Qs5gQJ3pEmrZd6cJ\" value:\"5000000000000000000000000\"  address:\"n1SAQy3ix1pZj8MPzNeVqpAmu1nCVqb5w8c\" value:\"5000000000000000000000000\"  address:\"n1SHufJdxt2vRWGKAxwPETYfEq3MCQXnEXE\" value:\"5000000000000000000000000\"  address:\"n1SSda41zGr9FKF5DJNE2ryY1ToNrndMauN\" value:\"5000000000000000000000000\"  address:\"n1TmQtaCn3PNpk4f4ycwrBxCZFSVKvwBtzc\" value:\"5000000000000000000000000\"  address:\"n1UM7z6MqnGyKEPvUpwrfxZpM1eB7UpzmLJ\" value:\"5000000000000000000000000\"  address:\"n1UnCsJZjQiKyQiPBr7qG27exqCLuWUf1d7\" value:\"5000000000000000000000000\"  address:\"n1XkoVVjswb5Gek3rRufqjKNpwrDdsnQ7Hq\" value:\"5000000000000000000000000\"  address:\"n1cYKNHTeVW9v1NQRWuhZZn9ETbqAYozckh\" value:\"5000000000000000000000000\"  address:\"n1dYu2BXgV3xgUh8LhZu8QDDNr15tz4hVDv\" value:\"5000000000000000000000000\" ]"
INFO[2018-10-31T13:50:11+08:00] Tail Block.                                   file=neblet.go func="neblet.(*Neblet).Setup" line=161 tail="{\"height\": 1, \"hash\": \"0000000000000000000000000000000000000000000000000000000000000000\", \"parent_hash\": \"0000000000000000000000000000000000000000000000000000000000000000\", \"acc_root\": \"db2a692aa8e21ba3a65fb952f441c5b346db29b3d4d10a7530b024e0ffc27050\", \"timestamp\": 0, \"tx\": 1, \"miner\": \"\", \"random\": \"\"}"
INFO[2018-10-31T13:50:11+08:00] Latest Irreversible Block.                    block="{\"height\": 1, \"hash\": \"0000000000000000000000000000000000000000000000000000000000000000\", \"parent_hash\": \"0000000000000000000000000000000000000000000000000000000000000000\", \"acc_root\": \"db2a692aa8e21ba3a65fb952f441c5b346db29b3d4d10a7530b024e0ffc27050\", \"timestamp\": 0, \"tx\": 1, \"miner\": \"\", \"random\": \"\"}" file=neblet.go func="neblet.(*Neblet).Setup" line=161
INFO[2018-10-31T13:50:11+08:00] Setuped Neblet.                               file=neblet.go func="neblet.(*Neblet).Setup" line=175
INFO[2018-10-31T13:50:11+08:00] Starting Neblet...                            file=neblet.go func="neblet.(*Neblet).Start" line=201
```

由于我们只启动了21个矿工节点中的一个矿工节点，所以每隔15*21s才出一个块。你可以启动更多的矿工节点，填补的空缺。但是需要注意，多个节点间的端口号不要相互冲突了。

