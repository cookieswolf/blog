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

- [一日一学_Go 语言 mgo（mongo 场景应用）](https://juejin.im/entry/58ef2399570c3500561c4b12)