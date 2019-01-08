# Ubuntu 18.04安装配置Shadowsocks实现科学上网

## 安装Shadowsocks 3.0.0

- 安装pip

```
sudo apt install python3-pip
```

- 安装Shadowsocks

```
sudo pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip -U
```

- 安装完后检查是否为3.0.0版本

```
sslocal --version
```

若显示Shadowsocks 3.0.0则进行下一步

## 配置Shadowsocks

- 编辑`shadowsocks`配置文件

```
sudo vim /etc/shadowsocks/config.json
```

配置如下

```
{
    "server":"45.32.43.123",
    "server_port":10538,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"wCOING9YUUR2sdfhI",
    "timeout":300,
    "method":"aes-256-gcm",
    "fast_open": false,
    "workers": 1,
    "prefer_ipv6": false
}
```

保存并退出

- 启动shadowsocks

```
setsid sslocal -c /etc/shadowsocks/config.json
```

## 通过`crontab`设置开机启动

- 配置`crontab`

```
sudo vim /etc/crontab
```

在最后一行添加如下命令

```
*/1 *  * * * root sslocal -c /etc/shadowsocks/config.json
```

保存并退出

- 重启`crontab`

```
/etc/init.d/cron restart
```

## Ubuntu配置网络

```
设置——>网络——>网络代理——>手动——>socks主机——>地址:127.0.0.1  端口:1080
```

保存打开`www.google.com`即可翻墙

也可将上诉配置方法配置到服务端，客户端去连接

## 参考资料

- [Ubuntu安装和使用shadowsocks](https://zhoujianshi.github.io/articles/2018/Ubuntu%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8shadowsocks/index.html)
- [科学上网 | Ubuntu使用shadowsocks翻墙](http://tanqingbo.com/2017/07/19/Ubuntu%E4%BD%BF%E7%94%A8shadowsocks%E7%BF%BB%E5%A2%99/)
- [Ubuntu 18.04安装配置Shadowsocks实现科学上网](https://leihungjyu.com/post/ubuntu-install-shadowsocks.html)