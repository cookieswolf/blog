# Ubuntu "Unable to locate package lrzsz"解决办法

首先尝试


```
sudo apt-get update
```

如果不能解决，则使用如下方法

```
$ sudo add-apt-repository main
$ sudo add-apt-repository universe
$ sudo add-apt-repository restricted
$ sudo add-apt-repository multiverse

$ sudo apt-get update
```

```
sudo apt-get install nodejs
sudo apt-get install npm
nodejs -v
```

参考文章

[Ubuntu "Unable to locate package lrzsz"解决办法](https://blog.csdn.net/menc15/article/details/78797996)

