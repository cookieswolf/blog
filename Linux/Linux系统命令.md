# Linux系统命令

- 查看Linux系统是Ubuntu还是CentOS或者其他系统

```
lsb_release -a
```

## Ubuntu端口转发

- 安装·rinetd

```
sudo apt-get install rinetd
```

- 配置`rinetd`

```
sudo vim /etc/rinetd.conf

// 在最后添加以下配置
0.0.0.0 27018 127.0.0.1 27017   // 将本机27018端口转发到27017

```

- 启动端口转发进程

```
rinetd -c /etc/rinetd.conf 
```

然后开放`27018`端口的防火墙服务即可通过27018访问27017端口

- error while loading shared libraries: libcurl.so.4

`error while loading shared libraries: libcurl.so.4: cannot open shared object file: No such file or directory`

解决方法:

```
apt-get install libcurl4-openssl-dev
```