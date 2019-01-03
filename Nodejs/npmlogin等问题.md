# NPM 常见使用方法

1. 404 Registry

npm login或者npm adduser时报如下错误`npm ERR! 404 Registry returned 404 for PUT on undefined`

**解决方法**

更换npm源

```
npm set registry https://registry.npmjs.org/
```

2. 更新npm至最新版本

```
npm install -g npm
```

3. npm降级

```
npm install npm@4 -g
```

4. 卸载npm

```
sudo npm uninstall npm -g
```
