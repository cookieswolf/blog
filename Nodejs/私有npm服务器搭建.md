# 私有npm服务器搭建

标签（空格分隔）： Node.js

---

# 私有npm服务器搭建

本次搭建是在`ubuntu`环境下搭建的,如果其他系统，将个别命令改成自己的


- 下载`cnpmjs.org`

```
# clone from github
$ git clone git://github.com/cnpm/cnpmjs.org.git $HOME/cnpmjs.org
$ cd $HOME/cnpmjs.org
```

- 下载安装`mysql`

```
sudo apt install mysql-server
```

中间会提示设置root 账户的密码
有的文章提到 还要 install mysql-client 现在不需要了，已经包含了

- 测试是否安装成功：

```
sudo netstat -tap | grep mysql
```

出现如下信息证明安装成功：

![](https://www.linuxidc.com/upload/2017_06/170615130620182.png)

- 进入`mysql`服务，创建数据库

```
# create mysql tables
$ mysql -u yourname -p
mysql> create database cnpmjs;
mysql> use cnpmjs;
mysql> source docs/db.sql;
```

- 添加个人配置

在`config`目录下创建`config.js`文件。(默认只有index.js文件)。在`config.js`中添加一下配置

```
 module.exports = {
    debug: false,
    enableCluster: true, // enable cluster mode
    enablePrivate: true, // enable private mode, only admin can publish, other use just can sync package from source npm
    database: {
		db: 'cnpmjstest',
        host: 'localhost',
        port: 3306,unknown database cnpmjs
        username: 'cnpmjs',
        password: 'cnpmjs123'  
    },
    admins: {
      admin: 'admin@cnpmjs.org',
    },
    syncModel: 'exist'// 'none', 'all', 'exist'
  };  
```

**注意**

* `enablePrivate`:设置为`true`时只能是管理员(`admins`里的)发布`cnpm`包。如果改为`false`则任何人都可以发布

- 运行`cnpm`

```
npm run start
```

- 测试运行结果

在本地访问`http://localhost:/7001`

显示

```
{"data_tables":{},"disk_size":0,"data_size":0,"index_size":0,"disk_format_version":0,"committed_update_seq":0,"update_seq":0,"purge_seq":0,"compact_running":false,"doc_count":0,"doc_del_count":0,"doc_version_count":0,"user_count":0,"sync_status":0,"need_sync_num":0,"success_sync_num":0,"fail_sync_num":0,"left_sync_num":0,"last_sync_time":0,"last_exist_sync_time":0,"last_sync_module":"","download":{"today":0,"thisweek":0,"thismonth":0,"lastday":0,"lastweek":0,"lastmonth":0},"db_name":"registry","instance_start_time":"1521614106548","node_version":"v4.2.6","app_version":"3.0.0-beta.1","donate":"https://www.gittip.com/fengmk2","sync_model":"exist","cache_time":1521615310970}
```

访问`http://localhost:7002`.显示`cnpm`界面。说明安装和运行成功

## 上传自己的npm包到私有`cnpm`服务器

- 安装`cnpm`客户端

```
sudo npm i cnpm -g
```

- 添加`cnpm`用户

```
cnpm adduser --registry=http://127.0.0.1:7001/
```

- 登录`cnpm`用户

```
cnpm login --registry=http://127.0.0.1:7001/
```

- 创建要上传的私有包

```
npm init test -y
```

- 修改`npm`包名为`cnpm`的

```
name:'test'  改为  name:'@cnpm/test'
```

- 发布`cnpm`包

```
cnpm publish --registry=http://127.0.0.1:7001/
```

## `nrm`管理多个`npm`源

- 安装`nrm`

```
sudo npm i nrm -g
```

- 添加本地`cnpm`服务为一个源

```
nrm add localnpm http://127.0.0.1:7001
```

- [使用CNPM搭建企业内部私有的NPM库](http://www.16boke.com/article/detail/155)
- [cnpmjs.org](https://github.com/cnpm/cnpmjs.org/wiki/Deploy)
- [企业私有 npm 服务器](https://www.jianshu.com/p/659fb418c9e3)
- [MYSQL 命令行大全 (简洁、明了、全面)](http://blog.csdn.net/jin13277480598/article/details/52504592)
- [在5分钟内搭建企业内部私有npm仓库](https://github.com/jaywcjlove/handbook/blob/master/CentOS/%E5%9C%A85%E5%88%86%E9%92%9F%E5%86%85%E6%90%AD%E5%BB%BA%E4%BC%81%E4%B8%9A%E5%86%85%E9%83%A8%E7%A7%81%E6%9C%89npm%E4%BB%93%E5%BA%93.md)








