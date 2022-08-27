<a name="TeNgT"></a>
## 前言
zookeeper是分布式协调一致性解决方案，是分布式应用程序的高性能协调服务。本篇不涉及理论知识，只构建基础开发使用环境。
<a name="CfqQm"></a>
## 环境准备
> 基础环境

- VMware Linux CentOS7虚拟机三台
- Java运行环境,文档基于JDK1.8安装
- [zookeeper安装包下载](https://downloads.apache.org/zookeeper/zookeeper-3.8.0/)，本文档基于zookeeper-3.8.0版本安装

![zk安装包.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1658244105595-0cbd9862-e5a7-411c-9ce6-08c2724a1291.png#clientId=uc3b8ea52-164d-4&crop=0&crop=0.2919&crop=1&crop=1&from=ui&height=375&id=uc6348eb6&margin=%5Bobject%20Object%5D&name=zk%E5%AE%89%E8%A3%85%E5%8C%85.png&originHeight=375&originWidth=878&originalType=binary&ratio=1&rotation=0&showTitle=false&size=27280&status=done&style=stroke&taskId=uc40c0936-787e-4476-ae41-5e6f84fed90&title=&width=878)
<a name="bEsQA"></a>
## 集群环境搭建
> 本次搭建使用三台主机，配置相应只对应一台主机，其余可参照重复配置

1. 上传并解压安装包，创建后续文件归档目录。三台机器重复此操作
```bash
#解压
[root@base install]# tar -zxvf apache-zookeeper-3.8.0-bin.tar.gz

#重命名文件夹
[root@base install]# mv apache-zookeeper-3.8.0-bin ./zookeeper

#进入文件解压目录
[root@base install]# cd zookeeper/

#创建数据＆日志文件夹(也可不创建,后面修改zookeeper配置文件后会自动创建)
[root@base zookeeper]# mkdir data logs
```

2. 配置主机环境变量
```bash
#配置环境变量
[root@base zookeeper]# vim /etc/profile

#添加环境变量，具体配置如图(因为机器之前添加过其它环境变量，配置时追加，使用:分割)
export ZK_HOME=/usr/local/install/zookeeper
export PATH=${JAVA_HOME}/bin:${ZK_HOME}/bin:$PATH

#保存配置并刷新环境变量，不报错则成功
[root@base zookeeper]# source /etc/profile
```
![zkev.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1658247000104-b69a3482-687b-4b02-8613-d2d7c41d8f6e.png#clientId=uba350039-a87c-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ud648de0e&margin=%5Bobject%20Object%5D&name=zkev.png&originHeight=288&originWidth=1037&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15977&status=done&style=none&taskId=ubd3896cf-ae8e-47c4-8ac1-d7bab25eee0&title=)

3. 修改配置文件
```bash
#进入安装目录conf文件夹下，重命名配置样本为zoo.cfg并进行修改
[root@base zookeeper]# cd conf/
[root@base conf]# mv zoo_sample.cfg zoo.cfg
#修改配置文件
[root@base conf]# vim zoo.cfg
```

4. 集群配置文件修改内容
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/install/zookeeper/data
#3.8中也可不配置，配置环境变量后，启动时bin/zkEnv.sh中会获取根路径并创建logs目录存储数据
dataLogDir=/usr/local/install/zookeeper/logs
# ?由个人配置的主机ip或者主机名替换，例：server.1=192.168.1.111:2288:3388
#server.X x需同后续设置的各主机myid一致
server.1=?:2288:3388
server.2=?:2288:3388
server.3=?:2288:3388
```

5. myid文件配置
> 注意：必须配置myid同集群各节点配置的数字对应，否则启动后集群链接会失败，
> 提示：Error contacting service. It is probably not running.

```bash
#各节点注意数字必须和对应主机配置文件的server.x中的x对应，比如
#server.1=192.168.1.111:2288:3388 则192.168.1.111这台主机的myid配置为1
[root@base zookeeper]# echo 1 > /usr/local/install/zookeeper/data/myid
```

6. 服务启动
```bash
#配置过环境变量可使用如下命令启动
[root@base data]# zkServer.sh start
#如果未配置环境变量，则要切换到安装路径的bin目录下
[root@base data]# cd /usr/local/install/zookeeper/bin
[root@base data]# ./zkServer.sh start
  ZooKeeper JMX enabled by default
  Using config: /usr/local/install/zookeeper/bin/../conf/zoo.cfg
  Starting zookeeper ... STARTED

#查看状态
[root@base data]# zkServer.sh status
  ZooKeeper JMX enabled by default
  Using config: /usr/local/install/zookeeper/bin/../conf/zoo.cfg
  Client port found: 2181. Client address: localhost. Client SSL: false.
  Mode: follower
```
<a name="evsNl"></a>
## 配置zookeeper集群环境开机自启动
<a name="N0Yy2"></a>
### 方式1：修改/etc/rc.d/rc.local文件
> 在/etc/rc.d/rc.local文件中添加JDK安装环境和/usr/local/install/zookeeper/bin/zkServer.sh start即可

```bash
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local

#添加JDK环境,根据自己情况配置
export JAVA_HOME=/usr/local/install/jdk
/usr/local/install/zookeeper/bin/zkServer.sh start
```
<a name="LQkqX"></a>
### 方式2：把zookeeper做成服务注册到开机启动列表中

1. 在/etc/rc.d/init.d目录下新建zookeeper脚本文件
```bash
[root@base data]# cd /etc/rc.d/init.d/
[root@base init.d]# touch zookeeper
```

2. 编写zookeeper开机自启动bash
```bash
#!/bin/bash
#chkconfig:2345 20 90
#description:zookeeper
#processname:zookeeper
export JAVA_HOME=/usr/local/install/jdk
case $1 in
        start) su root /usr/local/install/zookeeper/bin/zkServer.sh start;;
        stop) su root /usr/local/install/zookeeper/bin/zkServer.sh stop;;
        status) su root /usr/local/install/zookeeper/bin/zkServer.sh status;;
        restart) su /usr/local/install/zookeeper/bin/zkServer.sh restart;;
        *) echo "require start|stop|status|restart" ;;
esac
```

3. 脚本文件授权

`[root@base init.d]# chmod +x zookeeper`

4. 测试脚本是否有效并注册
```bash
# 测试脚本是否有效
[root@base init.d]# /etc/init.d/zookeeper start
ZooKeeper JMX enabled by default
Using config: /usr/local/install/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@base init.d]# /etc/init.d/zookeeper status
ZooKeeper JMX enabled by default
Using config: /usr/local/install/zookeeper/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower


# 查看服务
[root@base init.d]# chkconfig --list

#如果已存在zookeeper
[root@base init.d]# chkconfig --del zookeeper

# zookeeper开启自启
[root@base init.d]# chkconfig zookeeper on
```

5. 至此，集群部署完成，重启服务验证自启动。完结，后续相关知识体系及骚操作在系列专题补充。
<a name="NZ7Tq"></a>
## 扩展讨论
<a name="TGFXi"></a>
#### [关于为什么要修改conf/zoo_sample.cfg为zoo.cfg](https://zookeeper.apache.org/doc/r3.8.0/zookeeperStarted.html)
官方文档的说法是：为了方便讨论，规约说法。<br />`This file can be called anything, but for the sake of this discussion call it **conf/zoo.cfg**.`
<a name="byavL"></a>
#### zoo.cfg配置解读
```bash
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/usr/local/install/zookeeper/data
dataLogDir=/usr/local/install/zookeeper/logs
# the port at which the clients will connect
clientPort=2181
#cluster conf
server.1=192.168.5.101:2288:3388
server.2=192.168.5.102:2288:3388
server.3=192.168.5.103:2288:3388
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#跟RocketMQ一样可以使用普罗米修斯实现监控哦
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpHost=0.0.0.0
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true
```

- **tickTime** : the basic time unit in milliseconds used by ZooKeeper. It is used to do heartbeats and the minimum session timeout will be twice the tickTime.
- **dataDir** : the location to store the in-memory database snapshots and, unless specified otherwise, the transaction log of updates to the database.
- **clientPort** : the port to listen for client connections
<a name="vr0Eh"></a>
#### 集群监控
贴个官方方案，照着撸就行：<br />[https://zookeeper.apache.org/doc/r3.8.0/zookeeperMonitor.html](https://zookeeper.apache.org/doc/r3.7.0/zookeeperMonitor.html)


