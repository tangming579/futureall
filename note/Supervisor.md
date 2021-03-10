### 安装

```sh
yum install python-setuptools
easy_install pip
pip install supervisor
#生成配置文件supervisord.conf
mkdir /etc/supervisor
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```

supervisor安装完成后会生成三个执行程序：supervisortd、supervisorctl、echo_supervisord_conf，分别是supervisor的守护进程服务（用于接收进程管理命令）、客户端（用于和守护进程通信，发送管理进程的指令）、生成初始配置文件程序。

### 常用命令

```sh
##-c制定让其读取的配置文件
supervisord -c /etc/supervisor/supervisord.conf
#关闭supervisor
supervisorctl shutdown
#重新载入supervisor，在这里相当于重启supervisor服务，里面的服务也会跟着重新启动
supervisorctl reload
#添加/删除 要管理服务
supervisorctl update
```



```
reread ;重新加载配置文件
update ;将配置文件里新增的子进程加入进程组，如果设置了autostart=true则会启动新新增的子进程
status ;查看所有进程状态
status ;查看指定进程状态
start all; 启动所有子进程
start ; 启动指定子进程
restart all; 重启所有子进程
restart ; 重启指定子进程
stop all; 停止所有子进程
stop ; 停止指定子进程
reload ;重启supervisord
add ; 添加子进程到进程组
reomve ; 从进程组移除子进程，需要先stop。注意：移除后，需要使用reread和update才能重新运行该进程
```

### 配置说明

```
注：分号（;）开头的配置表示注释
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

;[program:xx]是被管理的进程配置参数，xx是进程的名称
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
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程
[include]
files = relative/directory/*.ini    ;可以指定一个或多个以.ini结束的配置文件
```

 