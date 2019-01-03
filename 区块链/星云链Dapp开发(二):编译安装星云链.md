# 星云链Dapp开发(二):编译安装星云链

- 下载源码

项目github地址:https://github.com/nebulasio/go-nebulas

```
cd $GOPATH/src/github.com/
mkdir nebulasio && cd nebulasio/
git clone https://github.com/nebulasio/go-nebulas.git
cd go-nebulas/
git checkout master
git checkout testnet
```

- 安装环境依赖

```
sudo apt-get update
sudo apt-get -y install build-essential libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
```

- 通过源码安装`rocksdb`

```
cd $GOPATH/src/github.com/nebulasio/go-nebulas
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

- 安装`vendor`依赖

```
cd $GOPATH/src/github.com/nebulasio/go-nebulas
wget http://ory7cn4fx.bkt.clouddn.com/vendor.tar.gz
tar xvf vendor.tar.gz
```

- 安装V8lib库

```
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make deploy-v8
```

- 编译

```
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make build
```

如果看到如下结果，说明编译成功

```
cd cmd/neb; go build -ldflags "-X main.version=1.1.0 -X main.commit=a656d32c0300ba96d4ea636a2c04ebd0e7007e48 -X main.branch=testnet -X main.compileAt=`date +%s`" -o ../../neb-a656d32c0300ba96d4ea636a2c04ebd0e7007e48
cd cmd/crashreporter; go build -ldflags "-X main.version=1.1.0 -X main.commit=a656d32c0300ba96d4ea636a2c04ebd0e7007e48 -X main.branch=testnet -X main.compileAt=`date +%s`" -o ../../neb-crashreporter
rm -f neb
ln -s neb-a656d32c0300ba96d4ea636a2c04ebd0e7007e48 neb
```

- 运行星云链节点

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