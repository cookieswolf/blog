# Ubuntu 下安装mongodb

* 机器型号:Ubuntu 16.04 LTS

- 导入包管理系统使用的公钥

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E162F504A20CDF15827F718D4B7C549A058F8B6B
```

- 为MongoDB创建一个列表文件

```
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.1 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list
```

- 重新加载本地包数据库

```
sudo apt-get update
```

- 安装MongoDB包

```
sudo apt-get install -y mongodb-org-unstable
```

- 常用命令

```
// 启动
sudo service mongod start

// 停止
sudo service mongod stop

// 重启
sudo service mongod restart
```

## 参考

- [官方文档](https://docs.mongodb.com/master/tutorial/install-mongodb-on-ubuntu/)
- [【全栈项目上线（vue+node+mongodb）】常见问题收集（持续更新）](https://segmentfault.com/a/1190000011782975)