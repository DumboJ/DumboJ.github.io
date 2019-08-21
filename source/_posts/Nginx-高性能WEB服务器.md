---
title: Nginx_高性能WEB服务器
date: 2019-08-17 23:47:38
tags:
 -Nginx
 -Web服务器
categories:
 -Nginx
cover: /static/bgpic/timg (1).jpg
---

### Nginx   高性能web服务器

#### 基础

##### 1.Nginx介绍

##### 2.Nginx安装启动

###### 安装包下载

[Nginx_Linux1.16.1]: http://nginx.org/download/nginx-1.16.1.tar.gz
[Nginx_windows1.16.1]: http://nginx.org/download/nginx-1.16.1.zip

###### 以Linux版本为例：

​	1.ssh登录阿里云CentOS7服务器

​	2.下载安装包

```
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

​	3.解压文件

```
tar -zxvf nginx-1.16.1.tar.gz 
```

​	4.进入文件目录，编译安装 <!--编译安装时可能遇到缺少库的情况，后面说明-->

```
./configure --prefix=/usr/local/projinstall/nginx
```

​	我安装时遇到的情景：

- ```
  ./configure: error: the HTTP gzip module requires the zlib library.
  You can either disable the module by using --without-http_gzip_module
  option, or install the zlib library into the system, or build the zlib library
  statically from the source with nginx by using --with-zlib=<path> option.
  系统中缺少zlib库
  
  解决--
  yum install -y zlib
  yum install -y zlib-devel
  ```

  

- ```
  error while loading shared libraries: libpcre.so.0: cannot open shared object file: No such file or directory
  系统中缺少pcre库
  
  解决--
  yum install -y pcre
  yum install -y pcre-devel
  ```

  编译无错误信息后安装

  ```
  make && make install
  ```

  5.启动Nginx

  ```
  cd /usr/local/projinstall/nginx
  ```

  | 目录 | 说明           |
  | ---- | -------------- |
  | conf | 配置文件       |
  | html | 网页文件       |
  | logs | 日志文件       |
  | sbin | 只要二进制程序 |

  ```
  cd ./sbin
  ls
  ./nginx		启动成功
  ```

  ​	注：若有端口占用情况，启动nginx服务器失败

  ```
  netstat -ant|p			--查看占用端口情况
  pkill -9  pid			--解除端口占用
  ./nginx					--再次启动服务
  ```

  ##### Nginx信号控制

  [官方地址]: https://www.nginx.com/resources/wiki/start/topics/tutorials/commandline/

| 信号参数  | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| TERM, INT | Quick shutdown                                               |
| QUIT      | Graceful shutdown                                            |
| KILL      | Halts a stubborn process                                     |
| HUP       | Configuration reload  --重新加载重读配置文件、                                                                                                                              Start the new worker processes with a new configuration、                                                         Gracefully shutdown the old worker processes |
| USR1      | Reopen the log files--日志切割：kill -USER1 pid  ，（重命名日志文件，再新建原名日志） |
| USR2      | Upgrade Executable on the fly       平滑的升级，当旧进程请求完毕则关闭 |
| WINCH     | Gracefully shutdown the worker processes                     |

##### Nginx命令控制：

| 命令                     | 说明                       |
| ------------------------ | -------------------------- |
| ./sbin/nginx  -s  reload | 配合文件修改重启加载       |
| ./sbin/nginx   -s  stop  | 结束进程                   |
| ./sbin/nginx             | 启动nginx                  |
| ./sbin/nginx  -t         | 确认修改的配置文件是否正确 |
| ./sbin/nginx  -s  reopen | 重新写入新的日志文件       |

​	

#### 应用部分

##### 1.Nginx虚拟主机配置	

```
user       www www;  ## Default: nobody
worker_processes  5;  ## Default: 1			工作子进程。一般为CPU*核心数，太大争夺CPU
error_log  logs/error.log;
pid        logs/nginx.pid;
worker_rlimit_nofile 8192;

events {
	#evert区，配置nginx连接的特性，一个work同时允许多少个连接
  worker_connections  4096;  ## Default: 1024		子进程最大允许连4096个连接	
}

#配置http服务器
http {
  include    conf/mime.types;
  include    /etc/nginx/proxy.conf;
  include    /etc/nginx/fastcgi.conf;
  index    index.html index.htm index.php;

  default_type application/octet-stream;
  log_format   main '$remote_addr - $remote_user [$time_local]  $status '
    '"$request" $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
  access_log   logs/access.log  main;
  sendfile     on;
  tcp_nopush   on;
  server_names_hash_bucket_size 128; # this seems to be required for some vhosts

  server { # php/fastcgi
    listen       80;
    server_name  domain1.com www.domain1.com;
    access_log   logs/domain1.access.log  main;
    root         html;

    location ~ \.php$ {
      fastcgi_pass   127.0.0.1:1025;
    }
  }

  server { # simple reverse-proxy
    listen       80;
    server_name  domain2.com www.domain2.com;
    access_log   logs/domain2.access.log  main;

    # serve static files
    location ~ ^/(images|javascript|js|css|flash|media|static)/  {
      root    /var/www/virtual/big.server.com/htdocs;
      expires 30d;
    }

    # pass requests for dynamic content to rails/turbogears/zope, et al
    location / {
      proxy_pass      http://127.0.0.1:8080;
    }
    
   server{
       listen		80;
       server_name	dumboj.cn
   }
  }

  upstream big_server_com {
    server 127.0.0.3:8000 weight=5;
    server 127.0.0.3:8001 weight=5;
    server 192.168.0.1:8000;
    server 192.168.0.1:8001;
  }

  server { # simple load balancing
    listen          80;
    server_name     big.server.com;
    access_log      logs/big.server.access.log main;

    location / {
      proxy_pass      http://big_server_com;
    }
  }
}
```

##### 2.Nginx日志切割

​	Nginx的配置文件中，server{access_log   logs/domain2.access.log  main;}

​	Nginx允许每个不同的虚拟主机server写入到不同的log文件

##### 3.Nginx与gzip设置

