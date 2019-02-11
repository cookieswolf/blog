# Mongodb副本集搭建

[TOC]

## 环境准备

服务器：centOS7
mongodb版本：3.6.2
副本集方案：1个主节点+2个二级节点
注意：部署生产环境的时候最好将不同的节点部署在不同的服务器上，本文为了演示，因此直接部署在同一台

## 配置

3个节点除了端口号其余的配置都一样，如下：

```
processManagement:
    fork: true
    pidFilePath: ../mongod.pid
net: 
    bindIp: 0.0.0.0
    port: 27019
    maxIncomingConnections: 65536  
    wireObjectCheck: true 
    ipv6: false
storage:
    dbPath: ../data
    indexBuildRetry: true
    journal:
        enabled: true
systemLog:
    path: ../logs/mongodb.log
    logAppend: true
    destination: file
replication:
    oplogSizeMB: 10240
    replSetName: myrepl
    secondaryIndexPrefetch: all
security:
    authorization: enabled
    clusterAuthMode: keyFile
    keyFile: ../../mongodb.key
    javascriptEnabled: true
```

## 步骤

1. 生成keyFile文件

```
openssl rand -base64 735 > mongodb.key  
chmod 600 mongodb.key
```

2. 分别启动各个节点mongodb实例

```
./mongod -f ../mongod.conf//进入mongodb的bin目录下
```

3. 服务器启动之后，进入任意一个节点的命令行，将三个的实例关联起来

```
> config = {
... _id : "myrepl",
... members : [
... {_id : 0, host : "127.0.0.1:27017"},
... {_id : 1, host : "127.0.0.1:27018"},
... {_id : 2, host : "127.0.0.1:27019"}]}
{
        "_id" : "myrepl",
        "members" : [
                {
                        "_id" : 0,
                        "host" : "127.0.0.1:27017"
                },
                {
                        "_id" : 1,
                        "host" : "127.0.0.1:27018"
                },
                {
                        "_id" : 2,
                        "host" : "127.0.0.1:27019"
                }
        ]
}
```

4. 初始化副本集的配置

```
> rs.initiate(config)
{
        "ok" : 1,
        "operationTime" : Timestamp(1520260635, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1520260635, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
> 
```

5. 当初始化配置信息后，可以明显的看到mongodb的命令行发生了变化，会显示出当前节点所属的副本集名称和节点类型。

```
[root@pmj bin]# ./mongo
MongoDB shell version v3.6.2
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.6.2
myrepl:PRIMARY>
```

至此，mongodb的副本集配置已经完成了，接下来是测试副本集是否可用

## 测试

1. 由于之前的配置是开启安全认证的，因此需要进入主节点先建一个root用户，然后认证

```
myrepl:PRIMARY> use admin
switched to db admin
myrepl:PRIMARY>  db.createUser({user: "admin",pwd:"admin",roles:[{role:"root",db:"admin"}]})
Successfully added user: {
        "user" : "admin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}
myrepl:PRIMARY> db.auth("admin","admin")
1
myrepl:PRIMARY>
```

2. 查看副本集状态

```
myrepl:PRIMARY> rs.config()
{
        "_id" : "myrepl",
        "version" : 1,
        "protocolVersion" : NumberLong(1),
        "members" : [
                {
                        "_id" : 0,
                        "host" : "127.0.0.1:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {
                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 1,
                        "host" : "127.0.0.1:27018",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {
                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "127.0.0.1:27019",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {
                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {
                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("5a9d561bdecc034b55b5a5b9")
        }
}
```

3. 查看主节点信息

```
myrepl:PRIMARY> rs.isMaster()
{
        "hosts" : [
                "127.0.0.1:27017",
                "127.0.0.1:27018",
                "127.0.0.1:27019"
        ],
        "setName" : "myrepl",
        "setVersion" : 1,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "127.0.0.1:27017",
        "me" : "127.0.0.1:27017",
        //.......................
}
```

4. 新建一个测试库mydb和测试用户test，用于测试数据插入

```
myrepl:PRIMARY> use mydb
switched to db mydb
myrepl:PRIMARY>  db.createUser({user: "test",pwd:"test",roles:[{role:"readWrite",db:"mydb"}]})
Successfully added user: {
        "user" : "test",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "mydb"
                }
        ]
}
myrepl:PRIMARY> use mydb
switched to db mydb
myrepl:PRIMARY> db.auth("test","test")
1
```

5. 插入100条数据

```
myrepl:PRIMARY> for(var i = 0; i < 100; i++) {
... db.testCollection.insert({order: i, name: "test" + i}) }
WriteResult({ "nInserted" : 1 })
myrepl:PRIMARY> db.testCollection.count()
100
```

6. 进入二级节点，查看数据是否同步

```
myrepl:SECONDARY> use mydb
switched to db mydb
myrepl:SECONDARY> db.auth("test","test")
1
myrepl:SECONDARY> db.testCollection.count()
2018-03-05T22:58:27.069+0800 E QUERY    [thread1] Error: count failed: {
        "operationTime" : Timestamp(1520261897, 1),
        "ok" : 0,
        "errmsg" : "not master and slaveOk=false",
        "code" : 13435,
        "codeName" : "NotMasterNoSlaveOk",
        //.........................省略
} 
```

当我们要查看二级节点数据时，发现出错，这是因为二级节点默认情况下是拒绝读取的，因此需开启读取功能

```
myrepl:SECONDARY> conn = new Mongo("127.0.0.1:27018")
connection to 127.0.0.1:27018
myrepl:SECONDARY> conn.setSlaveOk()
```

接着再查看数据，发现已经同步了

```
myrepl:SECONDARY> use mydb
switched to db mydb
myrepl:SECONDARY> db.auth("test","test")
1
myrepl:SECONDARY> db.testCollection.count()
100
```

## 故障转移

在这个例子中，副本集有3个成员，因此，我们将现在的主节点（端口号为27017）进程杀死，然后再查看副本集的状态，发现节点（端口号27019）的变为主节点

```
myrepl:PRIMARY> rs.isMaster()
{
        "hosts" : [
                "127.0.0.1:27017",
                "127.0.0.1:27018",
                "127.0.0.1:27019"
        ],
        "setName" : "myrepl",
        "setVersion" : 1,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "127.0.0.1:27019",
        "me" : "127.0.0.1:27019",
       //后面省略
```

测试数据插入也是正常的

```
myrepl:PRIMARY> use mydb
switched to db mydb
myrepl:PRIMARY> db.testCollection.count()
100
myrepl:PRIMARY> db.testCollection.insert({name:"test",pwd:"test",address:"beijing"})
WriteResult({ "nInserted" : 1 })
```

将最开始关掉进程的节点重启，会发现其变为副本集的二级节点，且数据会同步

## 增删节点

1. **删除节点**：进入主节点27019，然后移除27018节点

```
myrepl:PRIMARY> use admin
switched to db admin
myrepl:PRIMARY> db.auth("admin","admin")
1
myrepl:PRIMARY> rs.remove("127.0.0.1:27018")
{
        "ok" : 1,
        "operationTime" : Timestamp(1520340394, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1520340394, 1),
                "signature" : {
                        "hash" : BinData(0,"0VUzGmXGZ4EXAUC/HYJbRaPY6qc="),
                        "keyId" : NumberLong("6529469760260800513")
                }
        }
}
myrepl:PRIMARY> rs.isMaster()
{
        "hosts" : [
                "127.0.0.1:27017",
                "127.0.0.1:27019"
        ],
        "setName" : "myrepl",
        "setVersion" : 2,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "127.0.0.1:27019",
        "me" : "127.0.0.1:27019",
        //后面省略
```

2. **增加节点**:进入主节点27019，然后将刚刚删除的节点27018又重新加入到集群中

```
myrepl:PRIMARY> rs.add("127.0.0.1:27018")
{
        "ok" : 1,
        "operationTime" : Timestamp(1520340538, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1520340538, 1),
                "signature" : {
                        "hash" : BinData(0,"a3TQIbz8yCaDh4Uxh4yUUkF7auk="),
                        "keyId" : NumberLong("6529469760260800513")
                }
        }
}
myrepl:PRIMARY> rs.isMaster()
{
        "hosts" : [
                "127.0.0.1:27017",
                "127.0.0.1:27019",
                "127.0.0.1:27018"
        ],
        "setName" : "myrepl",
        "setVersion" : 3,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "127.0.0.1:27019",
        "me" : "127.0.0.1:27019",
        "electionId" : ObjectId("7fffffff0000000000000002"),
        后面省略.......
```

## 修改节点

### 修改优先级

1. 获取节点配置信息

```
myrepl:PRIMARY> cfg = rs.config()
{
        "_id" : "myrepl",
        "version" : 3,
        "protocolVersion" : NumberLong(1),
        "members" : [
                {
                        "_id" : 0,
                        "host" : "127.0.0.1:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "127.0.0.1:27019",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 3,
                        "host" : "127.0.0.1:27018",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
       //后面省略
```

2. 修改节点信息

```
myrepl:PRIMARY> cfg.members[0].priority=9
9
myrepl:PRIMARY> cfg.members[1].priority=6
6
myrepl:PRIMARY> cfg.members[2].priority=6
6
```

3. 重新配置副本集信息后，发现每个节点的priority值改了，并且优先级最高的变为了主节点

```
myrepl:PRIMARY> rs.reconfig(cfg)
{
        "ok" : 1,
        "operationTime" : Timestamp(1520341204, 2),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1520341204, 2),
                "signature" : {
                        "hash" : BinData(0,"G3PlqiS3AP7NHVVqp9svYSmNrdI="),
                        "keyId" : NumberLong("6529469760260800513")
                }
        }
}
myrepl:PRIMARY> rs.config()
{
        "_id" : "myrepl",
        "version" : 4,
        "protocolVersion" : NumberLong(1),
        "members" : [
                {
                        "_id" : 0,
                        "host" : "127.0.0.1:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 9,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "127.0.0.1:27019",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 6,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 3,
                        "host" : "127.0.0.1:27018",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 6,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
```

### 隐藏成员

要将节点变为隐藏节点，节点的priority值必须为0，否则配置会失败

```
myrepl:PRIMARY> cfg.members[2].priority=0
0
myrepl:PRIMARY> cfg.members[2].hidden=true
true
myrepl:PRIMARY> rs.reconfig(cfg)
{
        "ok" : 1,
        "operationTime" : Timestamp(1520341554, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1520341554, 1),
                "signature" : {
                        "hash" : BinData(0,"ZHXBdgARPnus/DDNl4fVjXVs30I="),
                        "keyId" : NumberLong("6529469760260800513")
                }
        }
}
myrepl:PRIMARY> rs.isMaster()
{
        "hosts" : [
                "127.0.0.1:27017",
                "127.0.0.1:27019"
        ],
        "setName" : "myrepl",
        "setVersion" : 5,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "127.0.0.1:27017",
        "me" : "127.0.0.1:27017",
```

### 延迟节点

要将节点变为延迟节点，节点的priority值必须为0，否则配置会失败

```
myrepl:PRIMARY> cfg.members[1].priority=0
0
myrepl:PRIMARY> cfg.members[1].slaveDelay=NumberLong(240)
NumberLong(60000)
myrepl:PRIMARY> rs.reconfig(cfg)
{
        "ok" : 1,
        "operationTime" : Timestamp(1520341773, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1520341773, 1),
                "signature" : {
                        "hash" : BinData(0,"6dkgGNTaykWIa0ojdBSqyZ7j8IM="),
                        "keyId" : NumberLong("6529469760260800513")
                }
        }
}
```

上面是将节点27019变为延迟节点，延迟时间为4分钟，现在往主节点数据库mydb的testHello集合插入一条数据

```
myrepl:PRIMARY> use mydb
switched to db mydb
myrepl:PRIMARY> db.testHello.insert({name:"hello",pwd:"world"})
WriteResult({ "nInserted" : 1 })
```

插入之后，在4分钟之内进入延迟节点，发现数据并没有同步，说明配置延迟节点成功

```
myrepl:SECONDARY> conn = new Mongo("127.0.0.1:27019")
connection to 127.0.0.1:27019
myrepl:SECONDARY> conn.setSlaveOk()
myrepl:SECONDARY> db = conn.getDB("mydb")
mydb
myrepl:SECONDARY> db.auth("test","test")
1
myrepl:SECONDARY> db.testHello.find()
myrepl:SECONDARY>
```

4分钟之后，再进入延迟节点查看，发现数据同步了

```
myrepl:SECONDARY> db.testHello.find()
{ "_id" : ObjectId("5a9e93f9bff707d8f99916fe"), "name" : "hello", "pwd" : "world" }
```

### 仲裁节点

现在副本集的组成为1个主节点+1个延迟节点+1个隐藏节点。现在将隐藏节点删除，再以添加仲裁节点的方式将此节点重新添加回来

```
myrepl:PRIMARY> rs.remove("127.0.0.1:27018")
{
        "ok" : 1,
        /............
}
myrepl:PRIMARY> rs.addArb("127.0.0.1:27018")
{
        "ok" : 1,
        //..............
        }
}
myrepl:PRIMARY> rs.config()
{
        "_id" : "myrepl",
        "version" : 14,
        "protocolVersion" : NumberLong(1),
        "members" : [
                {
                        "_id" : 0,
                        "host" : "127.0.0.1:27017",
                        //......
                },
                {
                        "_id" : 2,
                        "host" : "127.0.0.1:27019",
                        //......
                },
                {
                        "_id" : 3,
                        "host" : "127.0.0.1:27018",
                        "arbiterOnly" : true,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 0,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        //.................
}
```

## 参考文章

- [mongodb副本集搭建](https://www.jianshu.com/p/b5a7d13e1391)