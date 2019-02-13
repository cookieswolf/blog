# Mongodb使用笔记

## mongodb连接

```
const config = {
    url:"mongodb://47.99.34.144:27028/dappclub",
    options:{
        useNewUrlParser: true,  
        useCreateIndex: true,
        useFindAndModify: false,
        autoReconnect:true, //  当底层MongoDB驱动程序失去与MongoDB的连接时,它将自动尝试重新连接
        autoIndex: false, // Don't build indexes
        reconnectTries: Number.MAX_VALUE, // Never stop trying to reconnect
        reconnectInterval: 500, // Reconnect every 500ms
        poolSize: 10, // Maintain up to 10 socket connections
        // If not connected, return errors immediately rather than waiting for reconnect
        bufferMaxEntries: 0,
        connectTimeoutMS: 10000, // Give up initial connection after 10 seconds
        socketTimeoutMS: 45000, // Close sockets after 45 seconds of inactivity
        family: 4 // Use IPv4, skip trying IPv6
    }
}

mongoose.connect(config.url,config.options)
mongoose.model("Test",mongoose.Schema({username:String}))
```

参考:[Connections](https://mongoosejs.com/docs/connections.html)

## Mongodb高性能

- [MongoDB提升性能的18原则（开发设计阶段）](https://blog.fundebug.com/2018/09/19/18-principle-to-improve-mongodb-performance/)
- [大数据量下的数据库查询与插入如何优化？ （整理）](https://yq.aliyun.com/articles/4286)
- [MongoDB数据量大于2亿后遇到的问题 及原因分析](https://piaosanlang.gitbooks.io/mongodb/content/03day/shu-ju-liang-da-yu-dao-de-wen-ti.html)


## Mongodb配置副本集及分片

- [mongodb副本集搭建](https://www.jianshu.com/p/b5a7d13e1391)
- [mongodb分片集群搭建](https://www.jianshu.com/p/74e34e5c3516)

## 官方文档

- [配置文件选项](https://docs.mongodb.com/manual/reference/configuration-options/)
- [运行时数据库配置](https://docs.mongodb.com/manual/administration/configuration/)
- [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)
- [mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)
- [配置文件设置和命令行选项映射](https://docs.mongodb.com/manual/reference/configuration-file-settings-command-line-options-mapping/#conf-file-command-line-mapping)

- [MongoDB 通过配置文件启动](https://blog.csdn.net/zhu_tianwei/article/details/44261235)

## Mongoose
