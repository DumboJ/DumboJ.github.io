---

title: RocketMQ三主三从集群
date: 2022-04-4 09:34:42
tags:
 -RocketMQ
category:
 -RocketMQ
cover： static/bgpic/cata.jpg

---



<a name="jFxMr"></a>

## 前言

本文仅针对三主三从异步复制(async)模式的RocketMQ集群环境搭建，不涉及MQ相关概念和知识点内容讲解。仅为安装过程记录，可按需参考搭建环境。
<a name="LbGNI"></a>

## 环境准备

> 基础环境

- VMware Linux CentOS7虚拟机三台

- Java运行环境,文档基于JDK1.8安装

- [RocketMQ各版本源码和二进制文件下载](https://rocketmq.apache.org/dowloading/releases/)，本文档基于4.7.0二进制版本安装
  
  > 图形可视化相关

- [Apache rocketmq-export 项目](https://github.com/apache/rocketmq-exporter), 支持4.3.2以后版本的MQ

- [Prometheus](https://prometheus.io/download/)，本文档基于2.34.0

- Grafana（8.4.4版本,wget方式下载安装，相关内容查看后文）
  <a name="lrGnT"></a>
  
  ## 基础环境搭建
  
  <a name="bmJhY"></a>
  
  ### 单机运行环境
  
  > 以其中一台主机为例，其余类推，配置文件详情会在文档后面给出
1. 上传下载的RocketMQ二进制文件至服务器

2. 解压文件
   
   ```shell
   tar -zxvf rocketmq-all-4.7.0-bin-release.zip /usr/local/install
   mv rocketmq-all-4.7.0-bin-release rocketmq
   ```

3. 启动Name Server
   
   ```shell
   nohup sh bin/mqnamesrv &
   #查看日志是否启动成功
   tail -f ~/logs/rocketmqlogs/namesrv.log
   #启动成功
   The Name Server boot success...
   ```

```
4. 启动Broker
```shell
nohup sh bin/mqbroker -n localhost:9876 & 
#查看日志是否启动成功
tail -f ~/logs/rocketmqlogs/broker.log
#启动成功
The broker[%s, 192.168.5.101:10911] boot success...
#查看服务启动进程
jps
#表示单机环境nameserver和broker启动成功
962 NamesrvStartup
963 BrokerStartup
3765 Jps
```

5. 关闭服务
   
   ```shell
   #先关闭Broker
   sh bin/mqshutdown broker
   #关闭Name Server
   sh bin/mqshutdown namesrv
   ```
   
   <a name="HZShe"></a>
   
   ### Broker配置信息概览
   
   > 修改MQ安装目录下 conf/2m-2s-async中配置文件，初始会有4个文件，a/b各一份Master节点和Slaver节点,c节点更名即可。
   > 各主机对应配置文件参考表格内容，根据自己主机内容酌情更改

| 主机            | Master节点配置文件名       | Slaver节点配置文件名         |
| ------------- | ------------------- | --------------------- |
| 192.168.5.101 | broker-a.properties | broker-b-s.properties |
| 192.168.5.102 | broker-b.properties | broker-c-s.properties |
| 192.168.5.103 | broker-b.properties | broker-a-s.properties |

<a name="hEsNJ"></a>

### 配置详情

!!配置中同一台主机的主从节点配置文件端口号间隔大于1，否则源码处理机制会导致启动失败例：broker-a:9077|broker-b-s:9079
<a name="ShOJ9"></a>

#### 主机1

```shell
brokerClusterName=DefaultCluster
brokerName=broker-a
namesrvAddr=192.168.5.101:9876;192.168.5.102:9876;192.168.5.103:9876
# 0表示Master        >0 表示Slave
brokerId=0
#删除文件时间点 默认凌晨4点
deleteWhen=04
#文件保留时间 默认48小时
fileReservedTime=48
#发送消息时，自动创建服务器不存在的topic,默认创建的队列数
defaultTopicQueneNums=3
#是否允许 Broker 自动创建Topic 建议：线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许订阅组自动创建 建议：线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口，同一台主机上主从节点端口号最少间隔1
listenPort=9077
#CommitLog每个文件的默认大小1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存储30W条数据，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#文件磁盘最大利用率
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/home/data/rocketmq/store-a
#CommitLog存储路径
storePathCommitLog=/home/data/rocketmq/commitlog-a
#消费队列存储路径
storePathConsumeQueue=/home/data/rocketmq/consumequeue-a
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#destoryMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#Broker角色
#ASYNC_MASTER 异步复制Master
#SYNC_MASTER 同步双写Master
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

```shell
brokerClusterName=DefaultCluster
brokerName=broker-a
namesrvAddr=192.168.5.101:9876;192.168.5.102:9876;192.168.5.103:9876
# 0表示Master        >0 表示Slave
brokerId=1
#删除文件时间点 默认凌晨4点
deleteWhen=04
#文件保留时间 默认48小时
fileReservedTime=48
#发送消息时，自动创建服务器不存在的topic,默认创建的队列数
defaultTopicQueneNums=3
#是否允许 Broker 自动创建Topic 建议：线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许订阅组自动创建 建议：线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口，同一台主机上主从节点端口号最少间隔1
listenPort=9088
#CommitLog每个文件的默认大小1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存储30W条数据，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#文件磁盘最大利用率
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/home/data/rocketmq/store-b-s
#CommitLog存储路径
storePathCommitLog=/home/data/rocketmq/commitlog-b-s
#消费队列存储路径
storePathConsumeQueue=/home/data/rocketmq/consumequeue-b-s
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#destoryMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#Broker角色
#ASYNC_MASTER 异步复制Master
#SYNC_MASTER 同步双写Master
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

<a name="m4ijO"></a>

#### 主机2

```shell
brokerClusterName=DefaultCluster
brokerName=broker-b
namesrvAddr=192.168.5.101:9876;192.168.5.102:9876;192.168.5.103:9876
# 0表示Master        >0 表示Slave
brokerId=0
#删除文件时间点 默认凌晨4点
deleteWhen=04
#文件保留时间 默认48小时
fileReservedTime=48
#发送消息时，自动创建服务器不存在的topic,默认创建的队列数
defaultTopicQueneNums=3
#是否允许 Broker 自动创建Topic 建议：线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许订阅组自动创建 建议：线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口，同一台主机上主从节点端口号最少间隔1
listenPort=9077
#CommitLog每个文件的默认大小
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存储30W条数据，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#文件磁盘最大利用率
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/home/data/rocketmq/store-b
#CommitLog存储路径
storePathCommitLog=/home/data/rocketmq/commitlog-b
#消费队列存储路径
storePathConsumeQueue=/home/data/rocketmq/consumequeue-b
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#destoryMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#Broker角色
#ASYNC_MASTER 异步复制Master
#SYNC_MASTER 同步双写Master
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

```shell
brokerClusterName=DefaultCluster
brokerName=broker-c
namesrvAddr=192.168.5.101:9876;192.168.5.102:9876;192.168.5.103:9876
# 0表示Master        >0 表示Slave
brokerId=1
#删除文件时间点 默认凌晨4点
deleteWhen=04
#文件保留时间 默认48小时
fileReservedTime=48
#发送消息时，自动创建服务器不存在的topic,默认创建的队列数
defaultTopicQueneNums=3
#是否允许 Broker 自动创建Topic 建议：线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许订阅组自动创建 建议：线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口，同一台主机上主从节点端口号最少间隔1
listenPort=9088
#CommitLog每个文件的默认大小
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存储30W条数据，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#文件磁盘最大利用率
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/home/data/rocketmq/store-c-s
#CommitLog存储路径
storePathCommitLog=/home/data/rocketmq/commitlog-c-s
#消费队列存储路径
storePathConsumeQueue=/home/data/rocketmq/consumequeue-c-s
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#destoryMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#Broker角色
#ASYNC_MASTER 异步复制Master
#SYNC_MASTER 同步双写Master
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

<a name="SMoFF"></a>

#### 主机3

```shell
brokerClusterName=DefaultCluster
brokerName=broker-c
namesrvAddr=192.168.5.101:9876;192.168.5.102:9876;192.168.5.103:9876
# 0表示Master        >0 表示Slave
brokerId=0
#删除文件时间点 默认凌晨4点
deleteWhen=04
#文件保留时间 默认48小时
fileReservedTime=48
#发送消息时，自动创建服务器不存在的topic,默认创建的队列数
defaultTopicQueneNums=3
#是否允许 Broker 自动创建Topic 建议：线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许订阅组自动创建 建议：线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口，同一台主机上主从节点端口号最少间隔1
listenPort=9077
#CommitLog每个文件的默认大小
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存储30W条数据，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#文件磁盘最大利用率
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/home/data/rocketmq/store-c
#CommitLog存储路径
storePathCommitLog=/home/data/rocketmq/commitlog-c
#消费队列存储路径
storePathConsumeQueue=/home/data/rocketmq/consumequeue-c
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#destoryMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#Broker角色
#ASYNC_MASTER 异步复制Master
#SYNC_MASTER 同步双写Master
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

```shell
brokerClusterName=DefaultCluster
brokerName=broker-a
namesrvAddr=192.168.5.101:9876;192.168.5.102:9876;192.168.5.103:9876
# 0表示Master        >0 表示Slave
brokerId=1
#删除文件时间点 默认凌晨4点
deleteWhen=04
#文件保留时间 默认48小时
fileReservedTime=48
#发送消息时，自动创建服务器不存在的topic,默认创建的队列数
defaultTopicQueneNums=3
#是否允许 Broker 自动创建Topic 建议：线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 订阅组自动创建 建议：线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口，同一台主机上主从节点端口号最少间隔1
listenPort=9088
#CommitLog每个文件的默认大小
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存储30W条数据，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#文件磁盘最大利用率
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/home/data/rocketmq/store-a-s
#CommitLog存储路径
storePathCommitLog=/home/data/rocketmq/commitlog-a-s
#消费队列存储路径
storePathConsumeQueue=/home/data/rocketmq/consumequeue-a-s
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#destoryMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#Broker角色
#ASYNC_MASTER 异步复制Master
#SYNC_MASTER 同步双写Master
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

<a name="PJJkk"></a>

### 节点启动参数优化

> 本人虚拟机内存分配问题+测试数据量不大，所以更新每台主机MQ启动参数减少资源浪费

<a name="x0dcy"></a>

#### 修改Name Server启动配置参数

```shell
vim bin/runserver.sh
#默认配置
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
#更新为
JAVA_OPT="${JAVA_OPT} -server -Xms246m -Xmx256m -Xmn128m -XX:MetaspaceSize=64m -XX:MaxMetaspaceSize=128m"
```

<a name="b83Ps"></a>

#### 修改Broker启动配置参数

```shell
vim bin/runbroker.sh
#默认配置
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
#更新为
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```

<a name="nP3D9"></a>

### 配置集群各主机自启动

<a name="Gf7OX"></a>

#### 设置各主机MQ开机自启动

!!集群中的每台机器都要配置,各节点上启动properties文件名注意对应

```shell
vim /etc/rc.d/rc.local
#rocketmq start
ROCKETMQ_HOME=/usr/local/install/rocketmq
#启动Name Server
nohup sh $ROCKETMQ_HOME/bin/mqnamesrv >> $ROCKETMQ_HOME/logs/namesrv.log 2>&1 &
#启动Broker a主节点和b从节点
nohup sh $ROCKETMQ_HOME/bin/mqbroker -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-a.properties >> $ROCKETMQ_HOME/logs/broker-a.log 2>&1 &
nohup sh $ROCKETMQ_HOME/bin/mqbroker -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-b-s.properties >> $ROCKETMQ_HOME/logs/broker-b-s.log 2>&1 &
```

<a name="Gaw31"></a>

#### 授权重启主机

```shell
#授权rc.local
chmod +x /etc/rc.d/rc.local
#重启主机方式1
reboot
#重启主机方式2
shutdown -r now
#xshell链接到主机
ssh 192.168.5.101
```

> 坑：java运行环境即使在rc.local中配置指定.启动顺序也会导致配置不生效,自启动失败

启动日志报如下错误:

```shell
error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!
```

自启动问题处理:修改MQ安装bin目录下NameServer和Broker运行脚本

- 修改runserver.sh
  
  ```shell
  vim bin/runserver.sh
  #查找文件中如下位置修改JAVA_HOME为本机JDK安装目录
  [ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/local/install/jdk
  [ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/local/install/jdk
  ```

- 修改runbroker.sh
  
  ```shell
  vim bin/runbroker.sh
  #查找文件中如下位置修改JAVA_HOME为本机JDK安装目录
  [ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/local/install/jdk
  [ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/local/install/jdk
  ```
  
  <a name="xzSL5"></a>
  
  #### 重启验证集群各主机节点运行情况
  
  `ps -ef|grep rocketmq`
  
  > 如图所示:蓝色标注部分为后续图形化安装内容,不必关注,正常到这里会有三个进程-1个NameServer+2个broker

![微信图片_20220328232018.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648480921697-ee87b456-14c6-4b8b-ae2e-af2da76e2292.png#clientId=u44b12320-ecd7-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u33e3a589&margin=%5Bobject%20Object%5D&name=%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20220328232018.png&originHeight=545&originWidth=1695&originalType=binary&ratio=1&rotation=0&showTitle=false&size=112103&status=done&style=none&taskId=uea22e490-6248-44cd-9605-adbbff0c237&title=)

---

至此,MQ多节点异步复制集群环境就搭建成功了.依照文章中步骤安装配置有问题的还望指正并一起探讨深入.
<a name="sjRCn"></a>

## 可视化监控环境搭建

> 部署在一台机器上即可

<a name="gOXHY"></a>

### rocketmq-export部署

1. idea导入项目到本地
   
   > 导入地址:git@github.com:apache/rocketmq-exporter.git

![微信截图_20220329004757.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648486139452-4ce49b00-c42c-4f3d-90b3-2bfb0deff166.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u54af9e24&margin=%5Bobject%20Object%5D&name=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220329004757.png&originHeight=189&originWidth=660&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16331&status=done&style=none&taskId=u586c94bb-4376-4dbe-991a-df9c00c6d36&title=)<br />![微信截图_20220329005038.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648486260524-066ece42-bbc3-48d2-b5d3-ed52b642bf0f.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u0dd3592d&margin=%5Bobject%20Object%5D&name=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220329005038.png&originHeight=282&originWidth=998&originalType=binary&ratio=1&rotation=0&showTitle=false&size=21465&status=done&style=none&taskId=u26342101-96fe-4d74-8929-6161e194af6&title=)

2. 修改配置
   1. application.yml

![微信截图_20220329005737.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648486736682-5bb9049b-1ccc-47c4-8407-dec621b89bb2.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u9e254f21&margin=%5Bobject%20Object%5D&name=%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220329005737.png&originHeight=595&originWidth=1544&originalType=binary&ratio=1&rotation=0&showTitle=false&size=330196&status=done&style=none&taskId=ucdd8fed4-09dd-4a9a-8a5f-af3e0171802&title=)

2. pom.xml
   
   > 注释plugin,避免打包时失败

```bash
<plugin>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>2.17</version>
    <executions>
        <execution>
            <id>verify</id>
            <phase>verify</phase>
            <configuration>
                <configLocation>style/rmq_checkstyle.xml</configLocation>
                <encoding>UTF-8</encoding>
                <consoleOutput>true</consoleOutput>
                <failsOnError>true</failsOnError>
                 <includeTestSourceDirectory>false</includeTestSourceDirectory>
            </configuration>
            <goals>
                <goal>check</goal>
            </goals>
          </execution>
      </executions>
  </plugin>
```

> 修改MQ版本配置

```bash
<properties>
<!--修改rocketmq.version为个人安装版本-->
    <rocketmq.version>4.7.0</rocketmq.version>
    <docker.image.prefix>docker.io</docker.image.prefix>
</properties>
```

3. logback.xml

![logback.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648487577267-e329bf3f-7a43-408c-a221-eadd2ebd4331.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u2e192b1a&margin=%5Bobject%20Object%5D&name=logback.png&originHeight=650&originWidth=1539&originalType=binary&ratio=1&rotation=0&showTitle=false&size=379356&status=done&style=none&taskId=u6ee5ffd2-7aab-439f-b64d-88419a691ed&title=)

3. 打包上传
   
   ```bash
   #复制项目target目录下rocketmq-exporter-0.0.2-SNAPSHOT.jar上传至服务器
   mvn clean install
   ```

4. 发布
   
   ```bash
   nohup java -jar /usr/local/install/rocketmq/rocketmq-export/rocketmq-exporter-0.0.2-SNAPSHOT.jar &
   ```

5. 自启动
   
   1. 在/etc/rc.d/init.d目录下编写启动脚本并授权
      
      ```bash
      #切换目录
      cd /etc/rc.d/init.d
      #新建脚本文件
      touch rocketmq-export.sh
      #编写脚本
      vim rocketmq-export.sh
      ```
      
      脚本文件内容如下:修改jdk和jar包路径即可
      
      > #!/bin/sh
      > #chkconfig: 2345 80 90
      > #description:rocketmq-export 开机自启动脚本程序
      > 
      > #开启
      > cd /etc/rc.d/init.d
      > nohup /usr/local/install/jdk/bin/java -jar /usr/local/install/rocketmq/rocketmq-export/rocketmq-exporter-0.0.2-SNAPSHOT.jar &

授权为可执行文件

```bash
chmod +x rocketmq-export.sh
```

2. rc.local中引入脚本文件启动程序
   
   ```bash
   #修改rc.local文件
   vim /etc/rc.d/rc.local
   ```

#文件末尾插入内容
#rocketmq-export start
/bin/sh /etc/rc.d/init.d/rocketmq-export.sh

```
   3. 重启主机并验证脚本可用可能会有点小延时,可稍微等待一会儿查看
```bash
#查看端口号是否有对应服务
netstat -alnp |grep 5557
```

<a name="SJDBk"></a>

### Prometheus安装

1. 下载并上传文件至主机

[https://prometheus.io/download/](https://prometheus.io/download/)

2. 解压
   
   `tar -zxvf prometheus-2.34.0.linux-amd64.tar.gz`

3. 配置prometheus.yml中数据源
   
   ```bash
   #更新数据采集源
   vim prometheus-2.34.0.linux-amd64/prometheus.yml
   ```
   
   ![prome.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648490176067-6fbf3199-a2de-4169-8e06-2e862ef4608b.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ue6acb606&margin=%5Bobject%20Object%5D&name=prome.png&originHeight=662&originWidth=903&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42082&status=done&style=none&taskId=u7be7391f-09b7-4088-99f8-45ba312fe07&title=)

4. 启动服务

`./prometheus-2.34.0.linux-amd64/prometheus --config.file=prometheus-2.34.0.linux-amd64/prometheus.yml &`

5. 验证
   
   > 浏览器访问主机ip:9090,例192.168.5.101:9090 有相应指标提示就表示成功

![9090.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648490489105-104597ee-8582-4d9e-9209-9abefa1c7429.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u31996171&margin=%5Bobject%20Object%5D&name=9090.png&originHeight=725&originWidth=1628&originalType=binary&ratio=1&rotation=0&showTitle=false&size=96346&status=done&style=none&taskId=u7a4e97f8-304d-4703-89bf-57b73ce2cc3&title=)

6. 自启动
   
   > 编写/etc/rc.d/rc.local 写入执行命令,使用Prometheus安装绝对路径

```bash
#prometheus 自启动执行
./usr/local/install/rocketmq/rocketmq-export/prometheus-2.34.0.linux-amd64/prometheus --config.file=/usr/local/install/rocketmq/rocketmq-export/prometheus-2.34.0.linux-amd64/prometheus.yml &
```

<a name="zisfZ"></a>

### Grafana安装

1. 安装Granfana
   
   ```shell
   wget https://dl.grafana.com/enterprise/release/grafana-enterprise-8.4.4-1.x86_64.rpm
   sudo yum install grafana-enterprise-8.4.4-1.x86_64.rpm
   ```

2. 启动Grafana

`systemctl start grafana-server.service`<br />启动后浏览器访问 -- 主机ip:3000(默认初始登录用户名密码都为admin,登陆后会提示更改密码)<br />![grafana.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648491187707-bbc81892-5337-4963-a9e6-50906ed234f7.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u24597afc&margin=%5Bobject%20Object%5D&name=grafana.png&originHeight=741&originWidth=1913&originalType=binary&ratio=1&rotation=0&showTitle=false&size=100807&status=done&style=none&taskId=uf2f32a18-751f-4bbb-b1f0-0dbedd425f6&title=)

3. 配置Prometheus数据源

![ddd.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648491342100-74b96a1e-2902-4fee-bdfa-96a66a5da795.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u4019f3cd&margin=%5Bobject%20Object%5D&name=ddd.png&originHeight=751&originWidth=1883&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58188&status=done&style=none&taskId=u46c2db95-7c0f-4061-adb1-6c8a22086e2&title=)<br />![aa.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648491541515-e1915c25-4160-4f45-8dd1-c7f247aa5d59.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ueb15791a&margin=%5Bobject%20Object%5D&name=aa.png&originHeight=675&originWidth=938&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46179&status=done&style=none&taskId=u866c6169-77da-4c59-a752-4be764ed37a&title=)

4. 查找并配置Dashbroad面板
   
   > [https://grafana.com/search/?term=rocketmq&type=dashboard](https://grafana.com/search/?term=rocketmq&type=dashboard)

![query.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648492299769-f38022af-d4c5-4d44-8d03-20f9a19938e5.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u55fad8b5&margin=%5Bobject%20Object%5D&name=query.png&originHeight=835&originWidth=1461&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98951&status=done&style=none&taskId=ue46ebd6c-e9dd-4772-b058-661b44d1fca&title=)

> 选择模板并复制ID

![id.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648492368477-be980ebd-f1ad-43f5-8ccc-abeba6f12f95.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u219c7567&margin=%5Bobject%20Object%5D&name=id.png&originHeight=816&originWidth=1892&originalType=binary&ratio=1&rotation=0&showTitle=false&size=95989&status=done&style=none&taskId=u21d4c49e-afb1-4cbe-8fad-d73a4aff279&title=)

5. 配置模板并关联数据源

![ims.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648492827506-0aafdba0-efd0-4d4f-ba43-2d0a2004f4c2.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ub0ac1ba0&margin=%5Bobject%20Object%5D&name=ims.png&originHeight=624&originWidth=1872&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58377&status=done&style=none&taskId=u6771b12b-7c91-4929-9af3-9eab25c165e&title=)<br />![import.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648492562211-0bb28eb0-78c9-4324-a9f8-36d099f8601f.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u47263928&margin=%5Bobject%20Object%5D&name=import.png&originHeight=735&originWidth=1187&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28236&status=done&style=none&taskId=u8905c87d-f670-476d-8021-92830b71605&title=)

6. import

![finish.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648492897453-6fd28ddc-a7f0-4488-a995-52137fff72c8.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u88f002d0&margin=%5Bobject%20Object%5D&name=finish.png&originHeight=893&originWidth=1259&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74734&status=done&style=none&taskId=uee8b64ab-c1a7-4109-abfc-0ea8451df3c&title=)

7. 完成panel界面,模板panel可配置

![tu.png](https://cdn.nlark.com/yuque/0/2022/png/25928581/1648492979978-6f6a36ef-a8be-4d33-8ae2-3957c43bd08e.png#clientId=u1b20a0a4-4f21-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u6897037e&margin=%5Bobject%20Object%5D&name=tu.png&originHeight=922&originWidth=1917&originalType=binary&ratio=1&rotation=0&showTitle=false&size=122935&status=done&style=none&taskId=uc3dfd405-263a-4c74-a165-4989831bb35&title=)

8. Grafana服务自启动

`systemctl enable grafana-server.service`
<a name="EGACU"></a>

## 结语

Done!!<br />基础环境准备好了,后续根据官方文档资料开始深入研究实际使用场景高屋建瓴掌握MQ吧!<br /><(￣︶￣)↗[GO!]
