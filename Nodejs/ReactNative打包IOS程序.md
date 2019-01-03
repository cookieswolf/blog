# ReactNative 打包IOS应用程序

## 介绍

开发React Native的过程成,js代码和图片资源运行在一个`Debug Server`上，每次更新代码之后只需要使用`command+R`键刷新就可以看到代码的更改，这种方式对于调试来说是非常方便的。
但当我们需要发布App到App Store的时候就需要打包,使用离线的js代码和图片。这就需要把JavaScript和图片等资源打包成离线资源，在添加到Xcode中，然后一起发布到App Strore中。
打包离线资源需要使用命令`react-native bundle`(注：文中使用的项目名称为`RNIos`)

## 生成`bundle`文件

- 新建`bundle`目录

在`ios`目录下新建`bundle`目录

- 进入项目目录，运行以下打包命令

```
react-native bundle --entry-file index.js --platform ios --dev false --bundle-output ./ios/bundle/index.ios.jsbundle --assets-dest ./ios/bundle
```

- `--entry-file` ,ios或者android入口的js名称，比如`index.js`
- `--platform` ,平台名称(ios或者android)
- `--dev` ,设置为false的时候将会对JavaScript代码进行优化处理。
- `--bundle-output`, 生成的jsbundle文件的名称，比如`./ios/bundle/index.ios.jsbundle`
- `--assets-dest` 图片以及其他资源存放的目录,比如`./ios/bundle`

[详细命令](https://github.com/facebook/react-native/blob/master/local-cli/bundle/bundleCommandLineArgs.js)


- 在`package.json`中添加编译命令

```
{
    "scripts":{
        "bundle-ios":"node node_modules/react-native/local-cli/cli.js bundle --entry-file index.js  --platform ios --dev false --bundle-output ./ios/bundle/index.ios.jsbundle --assets-dest ./ios/bundle"
    }
}
```

以后打包直接运行`npm run bundle-ios`即可

## 在Xcode中集成

离线包生成完成之后，可以在ios目录下看到一个`bundle`目录，这个目录就是`bundle`生成的离线资源。
需要在Xcode中添加资源到项目中，必须使用`Create folder references`的方式添加文件夹.

- `Add Files to "RNIos"`

![](https://upload-images.jianshu.io/upload_images/22188-f509f0b987ef785d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/581)

- 选择`bundle`文件,在option中选择`Create folder references`

![](https://upload-images.jianshu.io/upload_images/22188-954ddb1a328133ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

- 添加到项目中的文件夹必须是蓝色

![](https://upload-images.jianshu.io/upload_images/22188-0ed8656c145cd170.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/535)

## 设置`AppDelegate.m`文件

修改AppDelegate.m中的加载包的方式（只需修改一次）,之后项目会自动跟据debug还是release状态加载不同的包

```
NSURL *jsCodeLocation;
#ifdef DEBUG
     //开发包
     jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];
#else
     //离线包
    jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"bundle/index.ios" withExtension:@"jsbundle"];
#endif
```

- 修改debug状态

将项目由debug状态改成release状态，`Xcode-->Product-->Scheme-->Edit Scheme...`

- 选择 Generic iOS Device ,修改Version和Build号
- clean一下项目，然后编译。此处生成.ipa文件的方式有多种，可以Archice,也可以先删除.app文件再在编译，将成的.app文件拖到iTunes里面生成。有了.ipa文件，接下来是上传app store还是内测就看你了

## 参考文章

- [React Native iOS详细打包步骤（2018.04更新）](https://www.jianshu.com/p/6d1ee919ded3)
- [React Native ios打包](https://www.jianshu.com/p/ce71b4a8a246)

