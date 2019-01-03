
# ReactNative run-android 问题

在MAC下，按照官方文档初始化一个android项目，最后run-android时报出以下错误。

```
SDK location not found. Define location with sdk.dir in the local.properties file or with an ANDROID_HOME environment variable
```

## 解决方案

- 查看`android sdk`路径

打开`android studio`,在`preference->Apperence & Behavior->System Settings->Android SDK`中找到sdk的路径

- 在`android`项目文件目录的根目录新建`local.properties`文件，填写一下内容

```
sdk.dir=/Users/username/Library/Android/sdk
```

保存即可

## 参考文章

- [SDK location not found. Define location with sdk.dir in the local.properties file or with an ANDROID_HOME environment variable](https://www.jianshu.com/p/63cd094f7143)