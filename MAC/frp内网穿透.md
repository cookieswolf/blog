# frp内网穿透

> 阿里云服务器 CentOS系统

- [frp项目地址](https://github.com/fatedier/frp/blob/master/README_zh.md)

## 服务器端

- 安装并配置frps

```
wget https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_linux_amd64.tar.gz
tar zxvf frp_0.20.0_linux_amd64.tar.gz
mv frp_0.20.0_linux_amd64.tar.gz frp
cd frp
```

- 编辑`frps.ini`

```
vim ./frps.init

# frps.ini
[common]
bind_port = 7000
vhost_http_port = 8080
```

退出并保存

- 启动服务端

```
./frps -c ./frps.ini

// 后台永久启动方式(关闭远程连接，进程还在)
nohup ./frps -c ./frps.ini &
或者
setsid ./frps -c ./frps.ini
```

## 客户端

- 下载客户端[frp MAC 64位](https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_darwin_amd64.tar.gz)

- 解压并编辑`./frpc.ini`

```
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000

[web]
type = http
local_port = 80
custom_domains = www.yourdomain.com
```

退出并保存

- 启动客户端

```
./frpc -c ./frpc.ini
```

显示一下信息说明连接成功

```
2018/07/21 00:37:09 [I] [control.go:246] [f585d7619cd7ad15] login to server success, get run id [f585d7619cd7ad15], server udp port [0]
2018/07/21 00:37:09 [I] [control.go:169] [f585d7619cd7ad15] [web] start proxy success
2018/07/21 00:37:09 [I] [control.go:169] [f585d7619cd7ad15] [ssh] start proxy success
```

**注意**

如果你是新买的阿里云主机。要检查一下几点配置

- 所用到的端口服务器端是否打开
- 在阿里云安全组规则配置中增加授权策略(包括协议类型和端口号。以及入和出)


## CentOS防火墙配置

```
- 停止防火墙
systemctl stop iptables.service

-  启动防火墙
systemctl start firewalld.service

- 开放端口
firewall-cmd --permanent --zone=public --add-port=80/tcp

- 重启服务使得改变端口操作起效
systemctl restart firewalld.service

- 检查端口是否有效启动
firewall-cmd --zone=public --query-port=80/tcp

```

报错`Failed to start firewalld.service: Unit firewalld.service is masked.`的解决办法

```
systemctl unmask firewalld.service
```


- [CentOS 7 yum 安装 Nginx](https://blog.csdn.net/u012486840/article/details/52610320)
- [Linux学习手册](https://linuxtools-rst.readthedocs.io/zh_CN/latest/base/index.html)