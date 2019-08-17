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

  





#### 应用部分

##### 1.Nginx虚拟主机配置

##### 2.Nginx日志切割

##### 3.Nginx与gzip设置

