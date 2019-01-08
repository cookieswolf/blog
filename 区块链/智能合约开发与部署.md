# 星云链Dapp开发(五):智能合约开发与部署

## 智能合约开发

### 智能合约存储

星云链智能合约(smart contract)提供了链上数据存储功能。类似于传统的key-value存储系统（eg:redis），可以付费（消耗gas）将数据存储到星云链上。

星云链的智能合约运行环境内置了存储对象`LocalContractStorage`，可以存储数字，字符串，JavaScript对象，存储数据只能在智能合约内使用，其他合约不能读取存储的内容。

**LocalContractStorage**的基本用法

- 存储

```
// 存储数据，数据会被json序列化成字符串保存
LocalContractStorage.put(key, value);
// 或者
LocalContractStorage.set(key, value);
```

- 读取

```
// 获取数据
LocalContractStorage.get(key);
```

- 删除

```
// 删除数据, 数据删除后无法读取
LocalContractStorage.del(key);
// 或者
LocalContractStorage.delete(key);
```

### 智能合约实例

下面通过实现一个非常简单的功能，来说明智能合约的开发。

功能简要说明
提交文字信息存储到星云链，包括：标题和内容
根据作者地址查看提交的信息
智能合约代码

```
'use strict';

// 定义信息类
var Info = function (text) {
    if (text) {
        var obj = JSON.parse(text); // 如果传入的内容不为空将字符串解析成json对象
        this.title = obj.title;                     // 标题
        this.content = obj.content;                 // 内容
        this.author = obj.author;                  // 作者
        this.timestamp = obj.timestamp;            // 时间戳
    } else {
        this.title = "";
        this.content = "";
        this.author = "";
        this.timestamp = 0;
    }
};

// 将信息类对象转成字符串
Info.prototype.toString = function () {
    return JSON.stringify(this)
};

// 定义智能合约
var InfoContract = function () {
    // 使用内置的LocalContractStorage绑定一个map，名称为infoMap
    // 这里不使用prototype是保证每布署一次该合约此处的infoMap都是独立的
    LocalContractStorage.defineMapProperty(this, "infoMap", {
        // 从infoMap中读取，反序列化
        parse: function (text) {
            return new Info(text);
        },
        // 存入infoMap，序列化
        stringify: function (o) {
            return o.toString();
        }
    });
};

// 定义合约的原型对象
InfoContract.prototype = {
    // init是星云链智能合约中必须定义的方法，只在布署时执行一次
    init : function () {

    },
    // 提交信息到星云链保存，传入标题和内容
    save : function (title, content) {
        title = title.trim();
        content = content.trim();

        if (title === "" || content === "") {
            throw new Error("标题或内容为空！");
        }

        if (title.length > 64) {
            throw new Error("标题长度超过64个字符！");
        }

        if (content.length > 256) {
            throw new Error("内容长度超过256个字符！");
        }
        // 使用内置对象Blockchain获取提交内容的作者钱包地址
        var from = Blockchain.transaction.from;
        // 此处调用前面定义的反序列方法parse，从存储区中读取内容
        var existInfo = this.infoMap.get(from);
        if (existInfo) {
            throw new Error("您已经发布过内容！");
        }

        var info = new Info();
        info.title = title;
        info.content = content;
        info.timestamp = new Date().getTime();
        info.author = from;

        // 此处调用前面定义的序列化方法stringify，将Info对象存储到存储区
        this.infoMap.put(from, info);
    },
    // 根据作者的钱包地址从存储区读取内容，返回Info对象
    read : function (author) {
        author = author.trim();
        if (author === "") {
            throw new Error("地址为空！");
        }
        // 验证地址
        if (!this.verifyAddress(author)) {
            throw new Error("输入的地址不存在！");
        }
        var existInfo = this.infoMap.get(author);
        return existInfo;
    },
    // 验证地址是否合法
    verifyAddress: function (address) {
        // 1-valid, 0-invalid
        var result = Blockchain.verifyAddress(address);
        return {
            valid: result == 0 ? false : true
        };
    }
};
// 导出代码，标示智能合约入口
module.exports = InfoContract;
```

## 合约部署

安装web钱包

下载地址：https://github.com/nebulasio/web-wallet

直接下载zip压缩包，在本地解压后点击文件夹中的“index.html”，即可使用

- 部署合约

![](./img/1.png)

将合约代码粘贴进去，然后选择钱包文件(我们在第三章发送交易的时候创建的账户，keystore文件保存在`$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`目录下)

![](./img/2.png)

- 查看合约部署状态

![](./img/3.png)


## 执行合约

- 执行`save`合约方法

![](./img/4.png)

- 执行`read`合约方法

![](./img/5.png)

至此，我们只能合约的开发、部署、测试都已完成
