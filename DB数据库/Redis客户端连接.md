# Redis客户端连接

修改`redis.conf`文件

## 设置

ubuntu 下在`/etc/redis/redis.conf`目录

1. 注释`bind`

```
bind 127.0.0.1
```

2. 设置密码

```
requirepass password
```

3. 关闭保护模式

```
protected-mode no
```

设置完以后重启服务 

```
src/redis-server ./redis.conf
```

4. 设置为后台守护进程启动

```
daemonize yes
```

## 测试

通过`redis-cli`来检查password是否设置成功

```
127.0.0.1:6379> auth password
ERR Client sent AUTH, but no password is set
```

这时其实password没有设置成功。我们可以直接通过`redis-cli`来设置

```
127.0.0.1:6379> config set requirepass password
OK
```

显示OK以后表示设置密码成功。这时再通过客户端连接即可

