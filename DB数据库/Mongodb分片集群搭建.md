# Mongodb分片集群搭建

[TOC]

## 环境

1. 服务器：centOS7
2. mongodb版本：3.6.2
3. 集群方案：路由服务器（2台）+分片服务器（2个分片，每个分片为一个副本集，每个副本集有3个节点）+配置服务器（1个副本集）

    - 路由服务器：127.0.0.1:27027、127.0.0.1:27028
    - 配置服务器：127.0.0.1:27022、127.0.0.1:27023、127.0.0.1:27024
    - 分片1：127.0.0.1:27017、127.0.0.1:27018、127.0.0.1:27019
    - 分片2：127.0.0.1:27010、127.0.0.1:27011、127.0.0.1:27012

## 配置

### 分片服务器

```
processManagement:
    fork: true
    pidFilePath: ../mongod.pid
net: 
    bindIp: 0.0.0.0
    port: 27010
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
sharding:
    clusterRole: shardsvr
    archiveMovedChunks: true
replication:
    oplogSizeMB: 10240
    replSetName: shard2
    secondaryIndexPrefetch: all
security:
    authorization: enabled
    clusterAuthMode: keyFile
    keyFile: ../../mongodb.key
    javascriptEnabled: true
```

### 配置服务器

```
processManagement:
    fork: true
    pidFilePath: ../mongod.pid
net: 
    bindIp: 0.0.0.0
    port: 27022
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
sharding:
    clusterRole: configsvr
    archiveMovedChunks: true
replication:
    oplogSizeMB: 10240
    replSetName: shardconf
    secondaryIndexPrefetch: all
security:
    authorization: enabled
    clusterAuthMode: keyFile
    keyFile: ../../mongodb.key
    javascriptEnabled: true
```

### 路由服务器

```
processManagement:
    fork: true
    pidFilePath: ../mongod.pid
net: 
    bindIp: 0.0.0.0
    port: 27028
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
   localPingThresholdMs: 15
sharding:
   configDB: shardconf/127.0.0.1:27022,127.0.0.1:27023,127.0.0.1:27024
security:
    authorization: enabled
    clusterAuthMode: keyFile
    keyFile: ../../mongodb.key
    javascriptEnabled: true
```

## 启动

启动顺序为：配置服务器 ------ 分片服务器 ------ 路由服务器。其中，配置服务器和分片服务器启动之后需要配置为副本集，因此配置分片集群前还需掌握副本集的配置，若不清楚的的可以参考这篇文章：https://www.jianshu.com/p/b5a7d13e1391

1. 启动配置服务器，并配置副本集

```
启动命令：mongod -f ../mongod.conf
[root@pmj bin]# netstat -apn|grep mongod
tcp        0      0 0.0.0.0:27022           0.0.0.0:*               LISTEN      3433/./mongod       
tcp        0      0 0.0.0.0:27023           0.0.0.0:*               LISTEN      3469/./mongod       
tcp        0      0 0.0.0.0:27024           0.0.0.0:*               LISTEN      3505/./mongod
```

2. 启动6台分片服务器，并配置为副本集

```
启动命令：mongod -f ../mongod.conf
[root@pmj bin]# netstat -apn|grep mongod
tcp        0      0 0.0.0.0:27017           0.0.0.0:*               LISTEN      3790/./mongod
tcp        0      0 0.0.0.0:27018           0.0.0.0:*               LISTEN      3944/./mongod       
tcp        0      0 0.0.0.0:27019           0.0.0.0:*               LISTEN      3971/./mongod          
tcp        0      0 0.0.0.0:27010           0.0.0.0:*               LISTEN      3917/./mongod       
tcp        0      0 0.0.0.0:27011           0.0.0.0:*               LISTEN      3818/./mongod       
tcp        0      0 0.0.0.0:27012           0.0.0.0:*               LISTEN      3845/./mongod       
```

3. 分别启动两个路由服务器

```
启动命令：mongos -f ../mongod.conf
[root@pmj bin]# netstat -apn|grep mongos
tcp        0      0 0.0.0.0:27027           0.0.0.0:*               LISTEN      4549/./mongos       
tcp        0      0 0.0.0.0:27028           0.0.0.0:*               LISTEN      4590/./mongos 
```

## 步骤

1. 随便进入一台路由服务器的admin库，创建一个用户

```
MongoDB server version: 3.6.2
mongos> use admin
switched to db admin
mongos> db.createUser({user: "routeadmin",pwd: "routeadmin",roles: [ { role: "userAdminAnyDatabase", db:"admin"}]})
Successfully added user: {
        "user" : "routeadmin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ]
}
```

2. 用新建的用户认证之后，又创建一个管理集群的用户

```
mongos> db.auth("routeadmin","routeadmin")
1
mongos> db.createUser({user: "routecluster",pwd: "routecluster",roles: [ { role: "clusterAdmin", db:"admin"}]})
Successfully added user: {
        "user" : "routecluster",
        "roles" : [
                {
                        "role" : "clusterAdmin",
                        "db" : "admin"
                }
        ]
}
```

3. 用新创建的管理集群的用户在admin进行认证，查看分片状态，发现此时集群还没加入分片

```
mongos> db.auth("routecluster","routecluster")
1
mongos> sh.status()
--- Sharding Status --- 
  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("5aaa8044c09e64eca284d025")
  }
  shards:
  active mongoses:
        "3.6.2" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours: 
                No recent migrations
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
```

4. 手动加入分片1和分片2副本

```
mongos> sh.addShard("shard2/127.0.0.1:27010,127.0.0.1:27011,127.0.0.1:27012")
{
        "shardAdded" : "shard2",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521192151, 8),
                "signature" : {
                        "hash" : BinData(0,"z0yS/XbgyvRCScGAa4gtTDWnhlw="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521192151, 8)
}
mongos> sh.addShard("shard1/127.0.0.1:27017,127.0.0.1:27018,127.0.0.1:27019")
{
        "shardAdded" : "shard1",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521192175, 7),
                "signature" : {
                        "hash" : BinData(0,"NnbSjs7/6VPYG9WOhjL35y69zh0="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521192175, 7)
}
```

5. 重新查看分片状态

```
mongos> sh.status()
--- Sharding Status --- 
  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("5aaa8044c09e64eca284d025")
  }
  shards:
        {  "_id" : "shard1",  "host" : "shard1/127.0.0.1:27017,127.0.0.1:27018,127.0.0.1:27019",  "state" : 1 }
        {  "_id" : "shard2",  "host" : "shard2/127.0.0.1:27010,127.0.0.1:27011,127.0.0.1:27012",  "state" : 1 }
  active mongoses:
        "3.6.2" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours: 
                No recent migrations
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
```

6. 开启数据库分片。新建一个mydb数据库并开启分片功能

```
mongos> sh.enableSharding("mydb")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521192251, 5),
                "signature" : {
                        "hash" : BinData(0,"bkeoAoAQHkMiFfrHEIleJFM4dj0="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521192251, 5)
}
```

7. 开启集合分片，注意：要开启的片键必须有索引。

```
mongos> sh.shardCollection("mydb.userinfo", {"_id": "hashed" } )
{
        "collectionsharded" : "mydb.userinfo",
        "collectionUUID" : UUID("58057dd4-0751-4de1-a04e-c0cde2285343"),
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521192520, 22),
                "signature" : {
                        "hash" : BinData(0,"PVhmZcNZei+yWvTA9b0bNJDgcaQ="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521192520, 14)
}
```

8. 切换成routeadmin用户并认证，然后新建一个测试用户，并用此用户插入数据

```
mongos> use mydb
switched to db mydb
mongos>  db.createUser({user: "test",pwd:"test",roles:[{role:"readWrite",db:"mydb"}]})
Successfully added user: {
        "user" : "test",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "mydb"
                }
        ]
}


mongos> db.auth("test","test")
1
mongos> for(var i = 0; i < 1000; i++) {
... db.userinfo.insert({name:"hello"+i, age:i})}
WriteResult({ "nInserted" : 1 })
```

9. 查看集合的分片状态，有两种

```
mongos> use admin
switched to db admin
mongos> db.auth("routecluster","routecluster")
1
mongos> sh.status()
--- Sharding Status ---
  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("5aab87e615963a28a6a00fac")
  }
  shards:
        {  "_id" : "shard1",  "host" : "shard1/127.0.0.1:27017,127.0.0.1:27018,127.0.0.1:27019",  "state" : 1 }
        {  "_id" : "shard2",  "host" : "shard2/127.0.0.1:27010,127.0.0.1:27011,127.0.0.1:27012",  "state" : 1 }
  active mongoses:
        "3.6.2" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours:
                1 : Success
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                shard1  1
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : shard1 Timestamp(1, 0)
        {  "_id" : "mydb",  "primary" : "shard1",  "partitioned" : true }
                mydb.userinfo
                        shard key: { "_id" : "hashed" }
                        unique: false
                        balancing: true
                        chunks:
                                shard1  2
                                shard2  2
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : NumberLong("-4611686018427387902") } on : shard1 Timestamp(2, 2)
                        { "_id" : NumberLong("-4611686018427387902") } -->> { "_id" : NumberLong(0) } on : shard1 Timestamp(2, 3)
                        { "_id" : NumberLong(0) } -->> { "_id" : NumberLong("4611686018427387902") } on : shard2 Timestamp(2, 4)
                        { "_id" : NumberLong("4611686018427387902") } -->> { "_id" : { "$maxKey" : 1 } } on : shard2 Timestamp(2, 5)



方式2：切换到具体的数据库，查看具体的集合  db.collection.stats()

mongos> use mydb
switched to db mydb
mongos> db.userinfo.stats()
{
        "sharded" : true,
        "capped" : false,
        "ns" : "mydb.userinfo",
        "count" : 1000,
        "size" : 53890,
        "storageSize" : 49152,
        "totalIndexSize" : 73728,
        "indexSizes" : {
                "_id_" : 32768,
                "_id_hashed" : 40960
        },
        "avgObjSize" : 53,
        "nindexes" : 2,
        "nchunks" : 4,
        "shards" : {
                "shard1" : {
                        "ns" : "mydb.userinfo",
                        "size" : 27101,
                        "count" : 503,
                        "avgObjSize" : 53,
                        "storageSize" : 24576,
                        省略......
                       
                },
                "shard2" : {
                        "ns" : "mydb.userinfo",
                        "size" : 26789,
                        "count" : 497,
                        "avgObjSize" : 53,
                        "storageSize" : 24576,
                        "capped" : false,
                         省略......
                }
        },
        "ok" : 1,
        省略....
}
```

## 修改

1. 切换主分片，注意：这是针对数据库而言的。形式为：`db.runCommand({movePrimary:<databaseName>, to:<newPrimaryShard>})`，如

```
mongos> db.runCommand({moveprimary:"mydb",to:"shard2"})
{
        "primary" : "shard2:shard2/127.0.0.1:27010,127.0.0.1:27011",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521255711, 7),
                "signature" : {
                        "hash" : BinData(0,"uHR/ZuXEHQF62+6I/oaBtx4jEx8="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521255711, 7)
}
```

切换完之后，再查看分片状态

```
mongos> sh.status()
--- Sharding Status --- 
  省略.....
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                shard1  1
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : shard1 Timestamp(1, 0) 
        {  "_id" : "mydb",  "primary" : "shard2",  "partitioned" : true }
                mydb.userinfo
                        shard key: { "_id" : "hashed" }
                        unique: false
                        balancing: true
                        chunks:
                                shard1  2
                                shard2  2
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : NumberLong("-4611686018427387902") } on : shard1 Timestamp(2, 2) 
                        { "_id" : NumberLong("-4611686018427387902") } -->> { "_id" : NumberLong(0) } on : shard1 Timestamp(2, 3) 
                        { "_id" : NumberLong(0) } -->> { "_id" : NumberLong("4611686018427387902") } on : shard2 Timestamp(2, 4) 
                        { "_id" : NumberLong("4611686018427387902") } -->> { "_id" : { "$maxKey" : 1 } } on : shard2 Timestamp(2, 5) 
```

2. 删除分片。删除分片前，请必须确保集群的均衡器已经开启，否则会造成此分片的数据丢失。

```
mongos>  sh.getBalancerState() 
true
```

查看集群的分片情况

```
mongos> db.adminCommand( { listShards: 1 } )
{
        "shards" : [
                {
                        "_id" : "shard2",
                        "host" : "shard2/127.0.0.1:27010,127.0.0.1:27011",
                        "state" : 1
                },
                {
                        "_id" : "shard1",
                        "host" : "shard1/127.0.0.1:27017,127.0.0.1:27018",
                        "state" : 1
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521256030, 3),
                "signature" : {
                        "hash" : BinData(0,"TQH8YLI0TUISws7h4i/OdxanNwk="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521256030, 3)
}
```

3. 开始删除。注意，若要删除的分片是mongodb的某个数据库的主分片，那么分片将无法删除，需要先改变数据库的主分片。删除的过程需要执行3个步骤：

```
mongos> db.adminCommand( { removeShard: "shard1" } )
{
        "msg" : "draining started successfully",
        "state" : "started",
        "shard" : "shard1",
        "note" : "you need to drop or movePrimary these databases",
        "dbsToMove" : [ ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521256228, 2),
                "signature" : {
                        "hash" : BinData(0,"WIOonn0iAx3B5cHb9O459YYORJo="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521256228, 2)
}
mongos> db.adminCommand( { removeShard: "shard1" } )
{
        "msg" : "draining ongoing",
        "state" : "ongoing",
        "remaining" : {
                "chunks" : NumberLong(3),
                "dbs" : NumberLong(0)
        },
        "note" : "you need to drop or movePrimary these databases",
        "dbsToMove" : [ ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521256230, 1),
                "signature" : {
                        "hash" : BinData(0,"DCjEzWMh5TJaqETj7LWf4wucOL0="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521256230, 1)
}
mongos> db.adminCommand( { removeShard: "shard1" } )
{
        "msg" : "removeshard completed successfully",
        "state" : "completed",
        "shard" : "shard1",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521256260, 2),
                "signature" : {
                        "hash" : BinData(0,"iC6blA3R0Uft0IhX1eWiAIrRSp0="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521256260, 2)
}
mongos> 
```

4. 检查分片是否删除成功

```
* 查看集群分片状态
mongos> db.adminCommand( { listShards: 1 } )
{
        "shards" : [
                {
                        "_id" : "shard2",
                        "host" : "shard2/127.0.0.1:27010,127.0.0.1:27011",
                        "state" : 1
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521256351, 1),
                "signature" : {
                        "hash" : BinData(0,"pRq/YI0eGRDXXfN+/6aY4y0HYDc="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521256351, 1)
}

* 查看数据库的数据分布情况
mongos> sh.status()
--- Sharding Status --- 
  省略.....
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                shard2  1
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : shard2 Timestamp(2, 0) 
        {  "_id" : "mydb",  "primary" : "shard2",  "partitioned" : true }
                mydb.userinfo
                        shard key: { "_id" : "hashed" }
                        unique: false
                        balancing: true
                        chunks:
                                shard2  4
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : NumberLong("-4611686018427387902") } on : shard2 Timestamp(3, 0) 
                        { "_id" : NumberLong("-4611686018427387902") } -->> { "_id" : NumberLong(0) } on : shard2 Timestamp(4, 0) 
                        { "_id" : NumberLong(0) } -->> { "_id" : NumberLong("4611686018427387902") } on : shard2 Timestamp(2, 4) 
                        { "_id" : NumberLong("4611686018427387902") } -->> { "_id" : { "$maxKey" : 1 } } on : shard2 Timestamp(2, 5) 

```

5. 重新添加从集合删除的旧分片，发现会出现错误，因为旧分片存在着脏数据，因此若要重新添加旧分片，需要将其数据删除

```
mongos> sh.addShard("shard1/127.0.0.1:27017,127.0.0.1:27018,127.0.0.1:27019")
{
        "ok" : 0,
        "errmsg" : "can't add shard 'shard1/127.0.0.1:27017,127.0.0.1:27018,127.0.0.1:27019' because a local database 'mydb' exists in another shard2",
        "code" : 96,
        "codeName" : "OperationFailed",
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521256621, 2),
                "signature" : {
                        "hash" : BinData(0,"nonD13wpVjZ3/pI20rEYikmu7V8="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521256621, 2)
}
```

6. 进入旧分片副本集的主节点，删除上述错误提示的数据库

```
shard1:PRIMARY> use mydb
switched to db mydb
shard1:PRIMARY> db.dropDatabase()
{
        "dropped" : "mydb",
        "ok" : 1,
        "operationTime" : Timestamp(1521258810, 2),
        "$gleStats" : {
                "lastOpTime" : {
                        "ts" : Timestamp(1521258810, 2),
                        "t" : NumberLong(4)
                },
                "electionId" : ObjectId("7fffffff0000000000000004")
        },
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521258810, 2),
                "signature" : {
                        "hash" : BinData(0,"04r8uymQzNMJf+jZxR8z193CYGc="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "$configServerState" : {
                "opTime" : {
                        "ts" : Timestamp(1521258786, 1),
                        "t" : NumberLong(2)
                }
        }
}
```

重新添加

```
mongos> sh.addShard("shard1/127.0.0.1:27017,127.0.0.1:27018,127.0.0.1:27019")
{
        "shardAdded" : "shard1",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1521258918, 2),
                "signature" : {
                        "hash" : BinData(0,"VwZ7jNBjCQ081UpYxZR5Ohjc8i4="),
                        "keyId" : NumberLong("6533465106343264276")
                }
        },
        "operationTime" : Timestamp(1521258918, 2)
}
```

数据库mydb自动均衡在各个分片上了

```
mongos> sh.status()
--- Sharding Status --- 
 省略.....
  databases:
        {  "_id" : "mydb",  "primary" : "shard2",  "partitioned" : true }
                mydb.userinfo
                        shard key: { "_id" : "hashed" }
                        unique: false
                        balancing: true
                        chunks:
                                shard1  2
                                shard2  2
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : NumberLong("-4611686018427387902") } on : shard1 Timestamp(5, 0) 
                        { "_id" : NumberLong("-4611686018427387902") } -->> { "_id" : NumberLong(0) } on : shard1 Timestamp(6, 0) 
                        { "_id" : NumberLong(0) } -->> { "_id" : NumberLong("4611686018427387902") } on : shard2 Timestamp(6, 1) 
                        { "_id" : NumberLong("4611686018427387902") } -->> { "_id" : { "$maxKey" : 1 } } on : shard2 Timestamp(2, 5) 
```



## 参考文章

- [mongodb分片集群搭建](https://www.jianshu.com/p/74e34e5c3516)
- [Mongodb分片集群部署](https://www.jianshu.com/p/cb55bb333e2d)
- [MongoDB分片集群搭建](http://blog.51cto.com/bigboss/2160311)