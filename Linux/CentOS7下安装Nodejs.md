# CentOS7上安装Nodejs

## 方式一

Ubuntu方式

```
sudo apt install nodejs
node -v
```

## 方式二

- 下载源码

```
cd /usr/local/src/
sudo wget https://npm.taobao.org/mirrors/node/v10.8.0/node-v10.8.0.tar.gz
```

- 解压源码,并重命名源码

```
sudo tar xvf node-v10.8.0.tar.gz
sudo mv node-v10.8.0.tar.gz nodejs
```

- 编译安装

```
cd nodejs/
sudo ./configure --prefix=/usr/local/node/10.8
sudo make
sudo make install
```

- 配置NODE_HOME，进入profile编辑环境变量

```
vim /etc/profile
```

- 设置nodejs环境变量，在 export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL 一行的上面添加如下内容:

```
export NODE_HOME=/usr/local/node/10.8
export PATH=$NODE_HOME/bin:$PATH
```

:wq保存并退出，编译/etc/profile 使配置生效

```
source /etc/profile
```

- 验证是否安装配置成功

```
node -v
```

# Ubuntu下安装Nodejs

```
sudo apt-get update
sudo apt-get install nodejs
sudo apt-get install npm
```

## Nodejs升级

```
npm install -g n
n latest  // 获取nodejs最新版本
```

## 扩展

- 通过以下命令可以查看到系统信息

```
lsb_release -a
```

原文地址:

- [通过包管理器方式安装 Nodejs](https://nodejs.org/zh-cn/download/package-manager/)
- [Node.js 安装配置](https://www.runoob.com/nodejs/nodejs-install-setup.html)
- [Nodejs源码](http://nodejs.cn/download/)
- [nodejs升级或版本切换](https://www.jianshu.com/p/22fe994edd72)
- [怎样用linux命令知道系统是ubuntu还是redhat或者其它的系统？](https://blog.csdn.net/dufufd/article/details/75330595)

