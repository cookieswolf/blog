# 构建Node.js脚手架

## 前言

像我们熟悉的 vue-cli，react-native-cli 等脚手架，只需要输入简单的命令 vue init webpack project，即可快速帮我们生成一个初始项目。在实际工作中，我们可以定制一个属于自己的脚手架，来提高自己的工作效率。

## 脚手架的原理

generators是yeoman生态系统的积木,通过yo命令运行而为终端用户生产文件，yeoman-generator本质上是一个有着完整项目结构的模板 ， yo根据用户选择不同的generator生成不同的项目。

imweb-cli也是一个以模板为核心的脚手架， 通过定制不同的模板， 可以实现初始化项目以及为已有项目添加符合规范的文件片段，这一点在新建页面的时候特别有用。 想了解的同学可以访问(https://www.npmjs.com/package/imweb-cli)

## 第三方库

- [commander.js](https://github.com/tj/commander.js)，可以自动的解析命令和参数，用于处理用户输入的命令。
- [download-git-repo](https://github.com/flipxfx/download-git-repo)，下载并提取 git 仓库，用于下载项目模板。
- [Inquirer.js](https://github.com/SBoudrias/Inquirer.js)，通用的命令行用户界面集合，用于和用户进行交互。
- [handlebars.js](https://github.com/wycats/handlebars.js)，模板引擎，将用户提交的信息动态填充到文件中。
- [ora](https://github.com/sindresorhus/ora)，下载过程久的话，可以用于显示下载中的动画效果。
- [chalk](https://github.com/chalk/chalk)，可以给终端的字体加上颜色。
- [log-symbols](https://github.com/sindresorhus/log-symbols)，可以在终端上显示出 √ 或 × 等的图标。

## 初始化项目

- 以`create-demo`为例

- 处理命令行

```
npm install commander download-git-repo inquirer handlebars ora chalk log-symbols --save-dev
```


```
{
  "name": "create-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "bin": "./bin/index.js",   // 在这里加入构建工程入口文件
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "devDependencies": {
    "download-git-repo": "^1.1.0",
    "handlebars": "^4.0.12",
    "inquirer": "^6.2.0",
    "log-symbols": "^2.2.0",
    "ora": "^3.0.0",
    "chalk": "^2.4.1",
    "commander": "^2.19.0"
  }
  "author": "",
  "license": "ISC"
}
```

- 下载模板

`./bin/index.js`文件

```
#!/usr/bin/env node  // 注意，这一句很关键，一定要加上
const program = require('commander');
const download = require('download-git-repo');
const ora = require('ora');
const chalk = require('chalk');
const symbols = require('log-symbols');

const spinner = ora('start create fibos dapp')

program.version('1.0.0', '-v, --version')
       .command('init <name>')
       .action((name) => {
          spinner.start()
          download('direct:https://github.com/yuanjunliang/create-demo.git', 'test/tmp', {clone: true}, (err) => {
              if(err){
                spinner.fail('create fibos dapp failed')
              }else{
                spinner.succeed('create fibos dapp success')
              }
          })
       });
program.parse(process.argv);
```

把项目发布到npm，然后`npm install create-demo -g`全局安装。就可以通过下列命令创建一个项目了

```
create-demo init project_name
```

## 参考文档

- [使用 Node.js 开发简单的脚手架工具 ](https://github.com/lin-xin/blog/issues/27)
- [自制前端脚手架](http://imweb.io/topic/59ffc48c1f0e50753869bf91)