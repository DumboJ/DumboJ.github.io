---
title: aliyun maven仓库
date: 2019-09-21 09:34:42
tags:
 -maven仓库配置
category:
 -aliyun maven仓库
cover： static/bgpic/cata.jpg
---

maven安装后，由于是自建项目，不能连到公司搭建私服和中央仓库（内网），所以新加maven使用阿里云仓库构建。

##### 添加方式：

- ​	在mirrors中添加

  ```xml
      <mirror>
        <id>mirrorId</id>
        <mirrorOf>repositoryId</mirrorOf>
        <name>Human Readable Name for this Mirror.</name>
        <url>http://my.repository.com/repo/path</url>
      </mirror>
  ```

  按照如上的配置，添加阿里云仓库相应配置

  ```xml
  	<mirror>  
  		<id>nexus-aliyun</id>  
  		<mirrorOf>central</mirrorOf>    
  		<name>Nexus aliyun</name>  
  		<url>http://maven.aliyun.com/nexus/content/groups/public</url>  
  	</mirror> 
  ```

  

- 在profiles中添加

  -profiles

  -----------profile

  ---------------------repositories

  ```xml
  <repository>
      <id>nexus-aliyun</id>
      <name>Nexus aliyun</name>
      <layout>default</layout>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
      <snapshots>
          <enabled>false</enabled>
      </snapshots>
      <releases>
          <enabled>true</enabled>
      </releases>
  </repository>
  ```

  

  阿里云的云效可以申请私有仓库构建自己的Maven仓库，留空捣鼓后补充~

   