# Linux 下 Go 的安装、配置 、升级和卸载

# 1\. 手动安装 Go

由于大家使用的 Linux 版本不尽相同，也不见得是最新版本或需要版本的 Go 语言包，所以我们说一下如何手动安装指定版本。

*   1\. 下载 Go 发行版

从官方地址：[https://golang.org/dl/](https://golang.org/dl/) 上下载合适的 二进制发行版 (例如: go1.10.4.linux-amd64.tar.gz)：

```
wget https://dl.google.com/go/go1.10.4.linux-amd64.tar.gz

```

*   2\. 提取压缩包

提取压缩包到合适的目录 (例如: /usr/local)：

```
sudo tar -xzf go1.10.4.linux-amd64.tar.gz -C /usr/local

```

*   3\. 建立软链接

```
sudo ln -s /usr/local/go/bin/* /usr/bin/

```

可以运行如下命令，验证是否安装成功：

```
go version

```

正常输出则说明安装成功，同时可以检查版本是否安装正确。

# 2\. 设置 Go 开发环境

## 2.1 创建工作空间

Go 代码必须放在 **_工作空间_** 内。它其实就是一个目录，其中包含三个子目录：

*   src 目录包含 Go 的源文件，它们被组织成 **_包_** （每个目录都对应一个包），
*   pkg 目录包含 **_包_** 编译后生成的库文件，
*   bin 目录包含 **_包_** 编译后生成可执行程序。

可在合适的位置创建工作空间和子目录，实例如下：

```
mkdir -p $HOME/go-workspace/src
mkdir -p $HOME/go-workspace/pkg
mkdir -p $HOME/go-workspace/bin

```

## 2.2 配置环境变量

使用 vi 编辑环境变量配置文件 `$HOME/.bashrc` ：

```
sudo vim $HOME/.bashrc

```

进入编辑界面后 `Shift+G` 跳转至尾行，按 `o` 新插入一行，输入如下：

```
export GOROOT=/usr/local/go  #设置为go安装的路径，有些安装包会自动设置默认的goroot
export GOPATH=$HOME/go-workspace   #默认安装包的路径
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

```

之后按 _Esc_ 键，`: wq` 保存退出。使配置文件生效：

```
source $HOME/.bashrc　　#注：这里不要用sudo执行，sudo无该命令

```

可运行 `go env` 查看 gol 环境变量：

```
go env

```

正常输出则说明配置成功，同时可对环境变量设置进行校验：

# 3\. 测试 Go 源码实例

通过构建一个简单的程序来检查 Go 的安装是否正确，具体操作如下：

首先创建一个名为 `hello.go` 的文件，并将以下代码保存在其中：

```
package main

import "fmt"

func main() {
    fmt.Printf("hello, world\n")
}

```

接着通过 go 工具运行它：

```
go run hello.go

```

若看到了 “hello, world” 信息，那么 Go 已被正确安装。

# 4\. 卸载 Go

卸载 Go，其实就是将前面安装 Go 的东西全部删除：

*   1\. 删除 go 目录：

```
sudo rm -rf /usr/local/go

```

*   2\. 删除软链接：

```
sudo rm -rf /usr/bin/go

```

# 5\. 升级 Go 版本

升级 Go 版本其实就是:

1.  卸载之前安装的旧版本 Go，
2.  再安装新版本的 Go。

* * *

**参考文章：**

*   起步 - Go 编程语言: [http://docscn.studygolang.com/doc/install](http://docscn.studygolang.com/doc/install)
*   如何使用 Go 编程: [http://docscn.studygolang.com/doc/code.html](http://docscn.studygolang.com/doc/code.html)
*   Ubuntu16.04 下部署 golang 配置环境: [http://www.aweb.cc/article/detail/id/583.html](http://www.aweb.cc/article/detail/id/583.html)

原文地址:https://www.jianshu.com/p/f2a237de8f07