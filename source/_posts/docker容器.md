---
title: docker容器
date: 2020-08-13 10:29:04
tags:  docker容器创建、容器相关命令
cover: 
---

##### 1.1容器创建

###### 1.1.1交互方式创建容器

> docker run -it --name=容器名称   镜像名称：标签（laster可忽略）  /bin/bash

###### 1.1.2.守护式方式创建容器

> docker run -di --name=容器名称   镜像名称：标签（laster可忽略）

###### 1.1.3.进入容器

> docker exec -it  容器名称 /bin/bash

##### 1.2容器停止与启动、目录挂载

###### 1.2.1.停止容器

> docker stop 容器名称（容器ID）

###### 1.2.2.启动容器

> docker start 容器名称（容器ID）

###### 1.2.3.文件拷贝

```shell
文件拷贝到容器内：
docker cp 需要拷贝的文件或目录 容器名称：容器目录

docker cp 容器名称：容器目录   拷贝的目标文件或目录 
```

###### 1.2.4.目录挂载

```shell
创建容器加-V 宿主机目录：容器目录，将宿主机的目录和容器内的目录进行映射，就可以通过修改宿主机某个目录的文件去影响容器
ps:
docker run -di -v /usr/local/projectinstall:/usr/local/projectinstall
--name=centos3 docker.io/centos
```

###### 1.2.5.查看容器IP地址信息

> --- 查看容器运行的各种数据
>
> docker inspect  centos3容器名称（容器ID）
>
> ---查看IP地址
>
> docker inspect --format='{{.NetworkSettings.IPAddress}}'  centos1容器名称（容器ID）
>
> 

###### 1.2.6.删除容器

```shell
//删除容器
docker rm centos7A(容器名称/容器ID)
//删除镜像
docker rmi redis(镜像名称)
```

