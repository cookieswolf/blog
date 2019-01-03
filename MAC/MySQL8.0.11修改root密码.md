# MySQL8.0.11修改root密码

> 在MySQL 8.04前，执行：SET PASSWORD=PASSWORD(‘[新密码]’);但是MySQL8.0.4开始，这样默认是不行的。因为之前，MySQL的密码认证插件是“mysql_native_password”，而现在使用的是“caching_sha2_password”。

这也可以解决Navicat1251错误

```
use mysql;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
FLUSH PRIVILEGES;
```

新密码即要修改的密码

另一种修改密码的方式

```
mysqladmin -u root -p password 新密码
```

新密码即要修改的密码，回车之后会要求让输入原始密码，然后就修改了密码

但是在忘记密码之后需要怎么办，还没有找到办法

原文地址:https://blog.csdn.net/qq_38265784/article/details/80915098