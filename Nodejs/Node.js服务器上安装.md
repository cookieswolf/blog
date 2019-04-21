# Node.js服务器上安装

## Centos7 安装 Nodejs

### 1\. 使用 EPEL 安装

EPEL（Extra Packages for Enterprise Linux）企业版 Linux 的额外软件包，是 Fedora 小组维护的一个软件仓库项目，为 RHEL/CentOS 提供他们默认不提供的软件包。
先确认系统是否已经安装了`epel-release`包：

```
$ yum info epel-release

```

如果有输出有关`epel-release`的已安装信息，则说明已经安装，如果提示没有安装或可安装，则安装

```
$ sudo yum install epel-release

```

安装完后，就可以使用`yum`命令安装 nodejs 了，安装的一般会是较新的版本，并且会将`npm`作为依赖包一起安装

```
$ sudo yum install nodejs

```

安装完成后，验证是否正确的安装，`node -v`，如果输出如下版本信息，说明成功安装

```
v6.9.4

```

### 2\. 使用官方编译过的二进制数据包安装

进入官网的[下载链接](https://link.jianshu.com?t=https://nodejs.org/download/release/)，在列表中进入想要下载的版本链接，选择与下面链接类似的想要下载的版本（*-linux-x64.tar.gz），右击并复制下载链接。进入用户主目录，使用`wget`命令下载，把下载路径粘贴到命令后

```
$ wget https://nodejs.org/download/release/latest-v6.x/node-v6.10.0-linux-x64.tar.gz

```

下载完成后使用下面的命令解压到`/usr/local`目录并安装：

```
$ sudo tar --strip-components 1 -xzvf node-v* -C /usr/local

```

安装完成后就可以使用方法 **1** 相同的方式来验证安装

### 3\. 源码安装 Nodejs

使用源码安装和二进制数据包安装的区别在于，源码安装还需要把源码编译，然后才能安装
下载源码的方式与上面的方法类似，进入官网[下载页面](https://link.jianshu.com?t=https://nodejs.org/download/release/)，选择想要下载的版本（`node-v*.tar.gz`），获取到下载链接（与下面的链接类似），进入用户目录，把源码包下载下来：

```
$ wget https://nodejs.org/download/release/latest-v6.x/node-v6.10.0.tar.gz

```

下载完后，解压并进入解压后的目录

```
$ tar xzvf node-v* && cd node-v*

```

要编译源码需要安装 `gcc` 和 `gcc-c++`，可以先使用`yum info package_name`检查是否已经安装了这两个软件包，如果没有，则进行安装

```
$ sudo yum install gcc gcc-c++

```

安装后，运行`configure`文件 并 编译

```
./configure
make

```

编译的时间会比较长，如果不出意外，通常在 20 来分钟左右，所以要耐心的等待编译完成。编译完成后，使用下面命令安装

```
$ sudo make install

```

安装完后，使用同样的方式验证安装，至此，结束。

当然还可以选择，使用`nvm`（node version manage）进行安装并管理 node 版本，但它默认是安装在用户目录下面，要全局安装，使所有用户都能使用同一 node，则需要另外再做处理的。
就这三种方法而言，`EPEL`方式显然会比较轻松简单，一般用这种方式就好，如果想折腾下源码安装，也不复杂。

参考资料：

*   [How To Install Node.js on a CentOS 7 server](https://link.jianshu.com?t=https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-a-centos-7-server)


- [Centos7 安装Nodejs](https://www.jianshu.com/p/7d3f3fa056e8)