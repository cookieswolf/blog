# npm包发布流程

1. 注册npm账号

- 方式一:去[npm官网](https://www.npmjs.com/)注册
- 方式二:通过终端注册

```
$ npm adduser
Username: YOUR_USER_NAME
Password: YOUR_PASSWORD
Email: YOUR_EMAIL@domain.com
```

- 查看npm当前使用的用户

```
$ npm whoami
```

2. 在终端登录npm账号

```
$ npm login
```

3. 创建npm包

```
npm init packagename -y
```

在生成的`package.json`中修改配置信息


**必须带有的字段**

- `name` :包名(全部小写，没有空格，可以使用下划线或者横线)
- `version`: 版本

**其他内容**

- `author`:作者信息
- `main`:程序入口文件，一般都是 `index.js`
- `description`:描述信息，有助于搜索
- `keywords`:[] 关键字，有助于在人们使用 `npm search` 搜索时发现你的项目
- `scripts`:支持的脚本，默认是一个空的 test
- `license`:默认是 MIT
- `bugs`:当前项目的一些错误信息，如果有的话
- `dependencies`:在生产环境中需要用到的依赖
- `devDependencies`:在开发、测试环境中用到的依赖
- `repository`:代码仓库

4. 发布程序包到`npm`

- 将npm源切换为`https://registry.npmjs.org/`

```
npm set registry https://registry.npmjs.org/
```

- `cd`到项目所在目录.执行一下命令发布项目

```
npm publish
```

## `package.json`示例

```
{
  "name": "eosjscrypto",
  "version": "1.0.0",
  "description": "A javascript crypto package for eos",
  "main": "lib/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/yuanjunliang/eosjscrypto.git"
  },
  "keywords": [
    "eos",
    "private key",
    "public key",
    "sign",
    "aes",
    "encryption",
    "decryption",
    "hash"
  ],
  "author": "yuanjunlinag",
  "license": "ISC",
  "dependencies": {
    "eosjs": "^6.1.9"
  }
}

```

## 参考文章

- [package.json文件](http://javascript.ruanyifeng.com/nodejs/packagejson.html)
- [npm 与 package.json 快速入门](http://blog.csdn.net/u011240877/article/details/76582670)
- [手把手教你用npm发布一个包](https://www.jianshu.com/p/36d3e0e00157)