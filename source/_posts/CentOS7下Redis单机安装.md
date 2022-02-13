---
title: CentOS下redis单机安装
date: 2022-02-13 21:32:18
tags: redis
cover: static/bgpic/nessary.jpg
---

# 准备工作

- [x] [官网](https://download.redis.io/releases/)下载redis安装包-此文档基于redis6.2.6版本

！！建议使用源代码编译安装，Linux包管理器安装的通常可用的不是最新版本

- [x] 安装GCC环境​
  
  ```shell
  yum -y install gcc gcc-c++ kernel-devel
  #查看gcc
  gcc -v
  ```

- [x] xftp上传并解压、构建
  
  ```shell
  #解压文件 stable为稳定版本,redis 使用标准版本标记进行版本控制,偶数为稳定版本,6.2为稳定版本,详见官网介绍
  tar xvzf redis-stable.tar.gz
  cd redis-stable
  #构建
  make
  ```

- [x] 构建正确性检测（可选）
  
  ```shell
  #文件解压根目录下
  make test
  #make test 检测时可能由于tcl版本太低失败,如检测错误提示tcl版本过低可运行如下命令
  yum install tcl
  ```

- [x] redis安装
  
  ```bash
  make install
  #提示信息如下表示安装成功
  Hint: It's a good idea to run 'make test' ;)
  
    INSTALL redis-server
    INSTALL redis-benchmark
    INSTALL redis-cli
  ```

- [x] 客户端工具

[Another-Redis-Desktop-Manager](https://github.com/qishibo/AnotherRedisDesktopManager)<br />done​
<a name="BEJkz"></a>

# Redis单机模式

<a name="tlfSe"></a>

## 1.修改配置文件

！ !redis安装目录下的redis.conf

```bash
#守护进程模式启动服务
daemonize yes
#修改NETWORK模块下配置bind 接受所有ip类型的客户端连接
bind  * -::*
#redis6开始使用ssl限制保证连接安全，但该配置可配置，如果ssl开启时会忽略
requirepass youpwd
#关闭保护模式，否则代码链接会失败
protected-mode no
```

<a name="SRMTb"></a>

## 2.检查防火墙

```shell
#检查系统防火墙设置
systemctl status firewalld.service
#如果info中ACTIVE：active(running)，关闭防火墙
systemctl stop firewalld.service
```

<a name="kqzbO"></a>

## 3.启动

```shell
#{path}表示redis安装路径
/{path}/src/redis-server /{path}/redis.conf
```

<a name="CWZtJ"></a>

## 4.设置启动命令别名

```bash
#每次通过安装目录和文件夹启动redis比较麻烦可通过设置别名启动redis服务和客户端
#切换到根目录
cd /root
#查看根目录下是否有对应文件
cat .bashrc
vi bashrc
#在已经配置的别名下添加如下命令(对应别名和配置信息自定义，路径为redis安装路径)
alias redis='/usr/local/install/redis/src/redis-server /usr/local/install/redis/redis.conf'
alias rediscli='/usr/local/install/redis/src/redis-cli'
#保存退出
wq
#刷新
source ./.bashrc
```

<a name="lwRIb"></a>

## 5.配置自启动

创建redis脚本文件

```bash
# 编写自启动脚本
# 注意在 /etc/init.d 中 编写 redis 文件
[root@localhost src]# vi /etc/init.d/redis
```

**​**脚本内容

```bash
#!/bin/bash
#
# chkconfig: 2345 10 90  
# description: Start and Stop redis   
PATH=/usr/local/bin:/sbin:/usr/bin:/bin   
REDISPORT=6379  
EXEC=/usr/local/install/redis/src/redis-server   
REDIS_CLI=/usr/local/install/redis/src/redis-cli   
PIDFILE=/var/run/redis.pid   
CONF="/usr/local/install/redis/redis.conf"  
AUTH="dumboj123"  
case "$1" in   
        start)   
                if [ -f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is already running or crashed."  
                else  
                        echo "Starting Redis server..."  
                        $EXEC $CONF   
                fi   
                if [ "$?"="0" ]   
                then   
                        echo "Redis is running..."  
                fi   
                ;;   
        stop)   
                if [ ! -f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is not running."  
                else  
                        PID=$(cat $PIDFILE)   
                        echo "Stopping..."  
                       $REDIS_CLI -p $REDISPORT  SHUTDOWN    
                        sleep 2  
                       while [ -x $PIDFILE ]   
                       do  
                                echo "Waiting for Redis to shutdown..."  
                               sleep 1  
                        done   
                        echo "Redis stopped"  
                fi   
                ;;   
        restart|force-reload)   
                ${0} stop   
                ${0} start   
                ;;   
        *)   
               echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2  
                exit 1  
esac
```

授权开启

```shell
# 保存后，进入到 /etc/init.d 中
[root@base src]# cd /etc/init.d

# 查看文件权限
[root@base init.d]# ll
-rw-r--r--. 1 root root   148 2月  13 19:43 bashrc
-rw-r--r--. 1 root root    92 2月  13 19:40 dump.rdb
-rw-r--r--. 1 root root 18281 5月  22 2020 functions
-rwxr-xr-x. 1 root root 10611 9月   4 17:12 mysql
-rwxr-xr-x. 1 root root  4569 5月  22 2020 netconsole
-rwxr-xr-x. 1 root root  7928 5月  22 2020 network
-rw-r--r--. 1 root root  1160 10月  2 2020 README
-rw-r--r--. 1 root root  1742 2月  13 20:16 redis

# 修改 redis 文件权限
[root@localhost init.d]# chmod 755 /etc/init.d/redis

# 再次查看 redis 权限,redis变为绿色
-rwxr-xr-x. 1 root root  1742 2月  13 20:16 redis

# 测试脚本是否有效
[root@base init.d]# /etc/init.d/redis start
Starting Redis server...
Redis is running...

# 查看服务
[root@base init.d]# chkconfig --list

#如果已存在redis
[root@base init.d]# chkconfig --del redis

# redis开启自启
[root@base init.d]# chkconfig redis on
```

<a name="WRBgF"></a>

# 服务卸载

```bash
# 查看是否存在 redis
[root@base ~]# rpm -qa | grep redis
[root@base ~]# find / -name redis
/etc/rc.d/init.d/redis
/etc/selinux/targeted/active/modules/100/redis
/usr/local/install/redis

# 查看服务是否开启状态
[root@base ~]# ps -ef|grep redis
root        689      1  0 20:20 ?        00:00:02 /usr/local/install/redis/src/redis-server *:6379
root       1394   1372  0 20:53 pts/0    00:00:00 grep --color=auto redis

# 杀掉进程
[root@base ~]# kill -9 169

# 卸载 redis 服务
[root@base ~]# rm -rf /usr/local/install/redis
[root@base ~]# rm -rf /etc/selinux/targeted/active/modules/100/redis

#系统配置修改
##删除自启动脚本并移除配置
[root@base ~]# rm -rf /etc/init.d/redis
[root@base ~]# chkconfig --del redis

#删除别名配置
[root@base ~]# vi /root/.bashrc
[root@base ~]# source /root/.bashrc
```

done
