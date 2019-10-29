---
layout: post
title: "安装和使用守护进程--Supervisor"
subtitle: '安装和使用守护进程--Supervisor'
author: "liu kai"
header-style: text
tags:
  - Supervisor
  - 博客
---

##  安装和使用守护进程--Supervisor

### 起因

__** 在Linux上有时候需要开启一个阻塞进程来监听操作。当ssh连上服务器，直接运行一个阻塞进程，然后退出服务器时，这个阻塞进程也会跟着关闭。**__

### 解决办法

_**使用python开发的Supervisor守护进程。它可以很方便的监听、启动、停止、重启一个或多个进程。用Supervisor管理的进程，当一个进程意外被杀死，supervisort监听到进程死后，会自动将它重新拉起，很方便的做到进程自动恢复的功能，不再需要自己写shell脚本来控制。**_

### 安装配置Supervisor

#### Centos安装

`yum install epel-release`

`yum install -y supervisor`

#### Ubuntu

`apt-get install supervisor`

#### 配置

**在/etc/supervisor/目录下有个conf.d的文件夹和supervisord.conf配置文件。打开配置文件**

`vim supervisord.conf`

**我们可以看到**

[include]
files = /etc/supervisor/conf.d/*.conf

**意思是Supervisor在启动的时候会加载conf.d目录下所有的conf配置文件。**

### horizon守护进程配置参考

**laravel的horizon守护进程配置**

`cd /etc/supervisor/conf.d/`

`vim horizon.conf`

**填入以下内容**

[program:horizon]
process_name=%(program_name)s
command=php /home/wwwroot/www.guaosi.com/artisan horizon ; 阻塞进程执行的命令
autostart=true ; 阻塞进程是否跟着Supervisor一起开机自动
autorestart=true ; 阻塞进程被异常退出是否自动重启
user=www ; 由哪个用户执行阻塞进程的命令
redirect_stderr=true
stdout_logfile=/home/wwwroot/www.guaosi.com/storage/logs/horizon.log ; 阻塞进程打印到控制台的内容写到哪里

### 启动服务

#### Centos启动

`systemctl start supervisord`

#### Ubuntu启动

`supervisord -c /etc/supervisor/supervisord.conf`

### 开机自启

#### Centos开机自启

新建文件 `supervisord.service`

#supervisord.service

[Unit] 
Description=Supervisor daemon

[Service] 
Type=forking 
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf 
ExecStop=/usr/bin/supervisorctl shutdown 
ExecReload=/usr/bin/supervisorctl reload 
KillMode=process 
Restart=on-failure 
RestartSec=42s

[Install] 
WantedBy=multi-user.target

**将文件拷贝到/usr/lib/systemd/system/**

`cp supervisord.service /usr/lib/systemd/system/`

**启动服务**

`systemctl enable supervisord`

**验证一下是否为开机启动**

`systemctl is-enabled supervisord`

#### Ubuntu开机自启

**编辑/etc/rc.local文件**

`vi /etc/rc.local`

**在exit 0之前加入以下命令**

`/usr/local/bin/supervisord`

保存并退出
最后修改rc.local权限

`chmod +x /etc/rc.local`

### 常用命令

**重新启动配置中的所有程序**

`supervisorctl reload`

**启动某个进程(program_name=你配置中写的程序名称)**

`supervisorctl start program_name`

**查看正在守候的进程**

`supervisorctl`

**停止某一进程 (program_name=你配置中写的程序名称)**

`supervisorctl stop program_name`

**重启某一进程 (program_name=你配置中写的程序名称)**

`supervisorctl restart program_name`

**停止全部进程**

`supervisorctl stop all`

**注意：显示用stop停止掉的进程，用reload或者update都不会自动重启。**


## 注意

**被Supervisor守护的进程都是常驻内存的，即如果修改了被守护的进程的源码，需要重启对这个进程的守护才能生效，否则还是未修改前的。**

`supervisorctl restart program_name`

