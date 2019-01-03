# Ubuntu下进程守护

## 进程守护者Supervisor

- 安装`supervisor`

```
apt-get install supervisor
```

- 添加进程配置

不建议全都写在supervisord.conf文件中，建议每个进程写一个`*.conf`后缀配置文件放到`/etc/supervisor/conf.d/`目录下

在`/etc/supervisor/supervisord.conf`配置文件最后可以看到

```
[include]
files = /etc/supervisor/conf.d/*.conf
```

也就是每次启动的时候都会加载`/etc/supervisor/conf.d/`目录下的`*.conf`配置，那么我们可以在`/etc/supervisor/conf.d/`目录下为每个项目创建一个`*.conf`后缀的配置文件

我们创建`progress.conf`配置文件，然后添加一下配置

```
[program:golang-http-server]
command=/root/go/src/github.com/blockchain  // 需要守护的启动程序，此处的可执行程序为blockchain
autostart=true  
autorestart=true
startsecs=10
stdout_logfile=/var/log/simple_http_server.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/var/log/simple_http_server.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB


[supervisord]
```

- 启动`supervisord`

```
supervisord -c /etc/supervisor/supervisord.conf
```

- 停止进程

```
supervisorctl shutdown
```

**错误记录**

如果配置最后没有`[supervisord]`,可能会报如下错误
运行报错:`Error: .ini file does not include supervisord section`
解决方案:在配置最后添加`[supervisord]`即可

**配置文件参数说明**

`注:`分号（;）开头的配置表示注释

```
[unix_http_server]
file=/tmp/supervisor.sock   ;UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ;socket文件的mode，默认是0700
;chown=nobody:nogroup       ;socket文件的owner，格式：uid:gid

;[inet_http_server]         ;HTTP服务器，提供web管理界面
;port=127.0.0.1:9001        ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
;username=user              ;登录管理后台的用户名
;password=123               ;登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10           ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info                ;日志级别，默认info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ;pid 文件
nodaemon=false               ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024                  ;可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ;可以打开的进程数的最小值，默认 200

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ;通过UNIX socket连接supervisord，路径与unix_http_server部分的file一致
;serverurl=http://127.0.0.1:9001 ; 通过HTTP的方式连接supervisord

; [program:xx]是被管理的进程配置参数，xx是进程的名称
[program:xx]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run  ; 程序启动命令
autostart=true       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=tomcat          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/opt/apache-tomcat-8.0.35/logs/catalina.out
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程

;包含其它配置文件
[include]
files = relative/directory/*.ini    ;可以指定一个或多个以.ini结束的配置文件
```

## Ubuntu利用系统的crontab进行进程守护

- 编辑`/etc/crontab`

```
vim /etc/crontab
```

- 添加进程配置

```
*/1 *   * * *   root    /root/go/src/github.com/blockchain-browser/blockchain-browser  // 设置定时时间，每个1分组检查一次
```

- 重启`crontab`

```
/etc/init.d/cron restart   // 重启进程
```

想要停止守护的话，就将配置注释，然后重启`crontab`


- [Supervisor官方文档](http://supervisord.org/index.html)
- [在CentOS7上用Supervisor运行Golang守护进程](https://www.jianshu.com/p/7d7c00b220bf)
- [Ubuntu下Supervisor安装、配置和使用](https://my.oschina.net/u/2396236/blog/1594853)
- [Ubuntu 安装和使用 Supervisor（进程管理）](https://www.cnblogs.com/xishuai/p/ubuntu-install-supervisor.html)
- [Supervisord安装和启动程序](https://blog.csdn.net/orangleliu/article/details/41317887)
- [Supervisor 配置过程](https://www.cnblogs.com/alimac/p/5858234.html)
- [Supervisor安装与配置](https://blog.csdn.net/xyang81/article/details/51555473)
- [crontab 定时任务](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/crontab.html)