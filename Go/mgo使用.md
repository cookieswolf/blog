# mgo使用



## 查询

- 两张表关联查询并合并数据

```
selector := []bson.M{
    {
        "$lookup": bson.M{  // 通过user_id进行关联
            "from":         "Users",
            "localField":   "user_id",
            "foreignField": "user_id",
            "as":           "user_info",  // 关联表返回结果是数组赋值给user_info
        },
    },
    {
        "$replaceRoot": bson.M{  // 文档替换
            "newRoot": bson.M{
                "$mergeObjects": []interface{}{  // 文档合并
                    bson.M{"$arrayElemAt": []interface{}{"$user_info", 0}},
                    "$$ROOT",
                },
            },
        },
    },
    {  // 过滤有效字段
        "$project": bson.M{"github_login": 1, "user_id": 1, "post_id": 1},
    },
    {
        "$limit": 10,
    },
}

var m []bson.M
if err := postModule.Pipe(selector).All(&m); err != nil {  // 用Post表为主表去关联查询Users表
    common.ResponseErr(c, "search failed", err)
    return
}
```



## 参考文档

- [Go实战--golang中使用MongoDB(mgo)](https://blog.csdn.net/wangshubo1989/article/details/75105397)
- [一日一学_Go 语言 mgo（mongo 场景应用）](https://www.jianshu.com/p/13b7f4630670)
- [MongoDB（Golang）查询&修改](https://www.jianshu.com/p/b63e5cfa4ce5)
- [Go与MongoDB](https://www.jianshu.com/p/a0620aa81a25)
- [使用Golang和MongoDB构建 RESTful API](https://juejin.im/post/5b46bb0ce51d4519115ce914)