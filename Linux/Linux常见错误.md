# Linux常见错误

 - Q1
 
  `unable to resolve host iZ2ze4xswj2tx5xmtq0f5pZ`

解决:

```
// 查询主机名
cat /etc/hostname
// 然后添加到hosts 文件的localhost 后面
vim /etc/hosts
```

 - Q2.

`./mongod: error while loading shared libraries: libcurl.so.4: cannot open shared object file: No such file or directory`

解决方法

```
apt-get install libcurl4-openssl-dev
```

- Q3.

`./mongod: error while loading shared libraries: libnetsnmpmibs.so.30: cannot open shared object file: No such file or directory`

解决方法

```
apt-get install snmpd snmp snmp-mibs-downloader
```

- Q4

```
Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

解决方案

```
sudo rm /var/cache/apt/archives/lock

sudo rm /var/lib/dpkg/lock
```

- [ubuntu常见错误--Could not get lock /var/lib/dpkg/lock解决](https://www.jianshu.com/p/6e4f16cf6398)