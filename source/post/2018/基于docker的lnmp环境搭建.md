```toml
title = "基于docker的lnmp环境搭建"
slug = "基于docker的lnmp环境搭建"
desc = "基于docker的lnmp环境搭建"
date = "2018-08-12 15:36:39"
update_date = "2018-08-12 15:36:39"
author = "doozz"
thumb = ""
tags = ["docker","lnmp"]
```

为了方便lnmp环境的搭建，本文章将会从零开始，使用docker搭建一个完整的php-nginx-mysql的工作环境

#### Docker

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

安装docker

```toml
# 安装docker,CentOS 7
yum install docker

# 查看docker版本
docker -v

# 启动docker服务
service docker start

# 删除docker容器
service rm 5182e96772bf

# 删除docker镜像
service rmi centos
```

#### Nginx+PHP

首先拉取centos,因为我们的nginx+php是在centos的基础上搭建

```toml
# 拉取centos
docker pull centos

# 查看拉取的镜像
[root@VM_16_9_centos source]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    latest              5182e96772bf        5 days ago          200 MB

#使用centos镜像来运行容器
[root@VM_16_9_centos source]# docker run -itcentos /bin/bash 
root@5182e96772bf:/# 

#此时我们已经进入容器内，安装 nginx
root@5182e96772bf:/#  yum install nginx

#修改nginx配置
root@5182e96772bf:/#  vi /etc/nginx/nginx.conf
#删除  include /etc/nginx/default.d/*.conf;下面的location 跟 error_page

#安装 php7.2
#首先更新EPEL 
root@5182e96772bf:/# rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
root@5182e96772bf:/# rpm -Uvh https://centos7.iuscommunity.org/ius-release.rpm

#安装php7.2
root@5182e96772bf:/# yum install php72u php72u-cli php72u-mysqli php72u-json php72u-xml

#修改php-fpm配置
php-fpm.conf
user=nginx
group=nginx

#设置权限
root@5182e96772bf:/# chmod -R nginx /vir/lib/php/session
root@5182e96772bf:/# chmod -R nginx /vir/lib/php/php-fpm

#安装 supervisor
root@5182e96772bf:/# yum install supervisor

#配置启动项
root@5182e96772bf:/# vi /etc/supervisord.d/php.ini
[program:nginx]
command=/usr/sbin/nginx    -g "daemon off;"
process_name=%(program_name)s
numprocs=1
umask=022
priority=999
autostart=true
autorestart=true
startsecs=10
startretries=3
exitcodes=0,2
stopsignal=QUIT
stopwaitsecs=10
user=root
redirect_stderr=true
stdout_logfile=/var/log/nginx/nginx.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stdout_events_enabled=false
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
stderr_events_enabled=false

[program:php]
command=/usr/sbin/php-fpm -F
process_name=%(program_name)s
numprocs=1
umask=022
priority=999
autostart=true
autorestart=true
startsecs=10
startretries=3
exitcodes=0,2
stopsignal=QUIT
stopwaitsecs=10
user=root
redirect_stderr=true
stdout_logfile=/var/log/nginx/php.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stdout_events_enabled=false
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
stderr_events_enabled=false

#启动supervisor
root@5182e96772bf:/# supervisortl

#查看nginx,php 运行状态
root@5182e96772bf:/# supervisortl status
保存并退出容器

#查看容器 
[root@VM_16_9_centos source]# docker ps -a

#保存容器修改
[root@VM_16_9_centos source]# docker commit -a"centos" -m"v1" 9f2e51aaa80d centos

#创建 启动新容器 
[root@VM_16_9_centos source]# docker run -p 80:80 --name nginx -v /root/www:/usr/share/nginx/html -v /root/nginx/conf/wp.conf:/etc/nginx/conf.d/wp.conf -d centos:v3 supervisord -n

#查看新容器 
[root@VM_16_9_centos source]# docker ps
```

#### Mysql

```toml
# 拉取mysql
[root@VM_16_9_centos source]# docker pull mysql

# 运行容器 
[root@VM_16_9_centos source]# docker run -p 3306:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql 

#进入容器 
[root@VM_16_9_centos source]# docker exex -it mysql /bin/bash 

#连接mysql 
mysql -h127.0.0.1 -p3306 -uroot -p123456 这里的ip是服务器ip而非容器ip
```

#### 结语

以上，就是基于docker的lnmp环境搭建。你也可以把制作的镜像更新到docker的私有仓库上，方便下次使用。
