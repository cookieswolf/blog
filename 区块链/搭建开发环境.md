
# 星云链Dapp开发(一):搭建开发环境

为了避免在配置过程中出现环境或者网络问题导致一些依赖包或者lib库下载不下来，建议使用一台国外服务器(本教程使用香港服务器)

## Go语言环境安装

注:本机采用[Ubuntu Server 18.04.1 LTS](https://www.ubuntu.com/download/server) 版本

- 下载go源码

```
wget https://dl.google.com/go/go1.11.linux-amd64.tar.gz
```

- 将go解压到/usr/local目录下

```
sudo tar -zxvf go1.11.linux-amd64.tar.gz -C /usr/local
```

- 将`/usr/local/go/bin`目录添加至PATH环境变量

```
export PATH=$PATH:/usr/local/go/bin
```

- 检查Go语言环境是否安装成功

```
go env
```

如果出现如下结果，说明配置成功

```
GOARCH="amd64"
GOBIN="/home/yuanjunliang/go/bin"
GOCACHE="/home/yuanjunliang/.cache/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/home/yuanjunliang/go"
GOPROXY=""
GORACE=""
GOROOT="/usr/lib/go1.10"
GOTMPDIR=""
GOTOOLDIR="/usr/lib/go1.10/pkg/tool/linux_amd64"
GCCGO="gccgo"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build892316758=/tmp/go-build -gno-record-gcc-switches"
```

- 配置`GOPATH`工作目录

进入上述`go env`的返回结果中`GOPATH`所对应的目录,并创建相应的工作目录

注:`这里的具体目录根据个人真实情况而定`

```
mkdir /home/yuanjunliang/go
cd /home/yuanjunliang/go
mkdir bin pkg src src/github.com src/golang.org
```

至此，Go环境搭建成功