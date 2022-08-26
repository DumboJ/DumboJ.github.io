<a name="rzO5x"></a>
## 安装方法
按需选择不同的方式安装 Docker Engine：
> 存储库方式：便于安装和升级任务。也是本篇介绍并推荐使用的方法。

- 大多数用户 [设置 Docker 存储库](https://docs.docker.com/engine/install/centos/#install-using-the-repository)并从中安装
- 下载 RPM 包并 [手动安装它](https://docs.docker.com/engine/install/centos/#install-from-a-package)并完全手动管理升级。无法访问 Internet 的系统安装 Docker 等情况下；
- 在测试和开发环境中，一些用户选择使用自动化 [便利脚本](https://docs.docker.com/engine/install/centos/#install-using-the-convenience-script)来安装 Docker；
<a name="foPnJ"></a>
## 安装Yum工具包并设置存储库

1. 安装yum工具包

`[root@base ~]# sudo yum install -y yum-utils`

2. 执行`yum-config-manager`应用程序设置存储库源
- 官方存储库源（速度较慢）
```bash
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

- 阿里云存储库源（建议使用）
```bash
[root@base ~]# sudo yum-config-manager \
> --add-repo \
> http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

已加载插件：fastestmirror
adding repo from: http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
grabbing file http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```
<a name="QFucf"></a>
## 安装Docker引擎

- 安装特定版本的 Docker Engine：

---

   1. 查询绑定存储库源中并列出可用版本(示例按版本号从最高到最低对结果排序)，选择并安装
```bash
[root@base ~]# yum list docker-ce --showduplicates | sort -r
已加载插件：fastestmirror
已安装的软件包
可安装的软件包
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
 * extras: mirrors.aliyun.com
#存储库中可用版本的Docker Engine
docker-ce.x86_64            3:20.10.9-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.8-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.7-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.6-3.el7                    docker-ce-stable 
... ...
```
✍️拓展点
:::info
Docker Engine 版本号特定于当前系统的版本，例如本例为CentOS7（在本例中由.el7后缀表示）<br />可查看当前系统版本验证 <br />[root@base ~]# cat /proc/version<br />Linux version 3.10.0-1160.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) ) #1 SMP Mon Oct 19 16:18:59 UTC 2020
:::

---

   2. 通过完全限定的包名称安装特定版本命令格式

`$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io docker-compose-plugin`

✍️VERSION_STRING 说明<br />版本字符串截取规则为：（取a步骤中命令列出的第二列），从第一个冒号  : 开始，到第一个连字符 - 结束，即为 VERSION_STRING
> 例如要特定安装a步骤列出版本号中的如下版本的Docker Engine：
> docker-ce.x86_64            3:20.10.6-3.el7                    docker-ce-stable 
> 则 <VERSION_STRING>为 20.10.6

   3. 更换VERSION_STRING值执行安装
```bash
[root@base ~]# sudo yum install docker-ce-20.10.6 docker-ce-cli-20.10.6 containerd.io docker-compose-plugin

已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
docker-ce-stable                                                                                                                                          | 3.5 kB  00:00:00     
匹配 3:docker-ce-20.10.6-3.el7.x86_64 的软件包已经安装。正在检查更新。
匹配 1:docker-ce-cli-20.10.6-3.el7.x86_64 的软件包已经安装。正在检查更新。
软件包 containerd.io-1.6.7-3.1.el7.x86_64 已安装并且是最新版本
软件包 docker-compose-plugin-2.6.0-3.el7.x86_64 已安装并且是最新版本
无须任何处理
```
> P.S: 此命令会安装 Docker，但不会启动 Docker。它还会创建一个 docker组，但是默认情况下它不会将任何用户添加到该组中。

- 安装最新版本的 Docker Engine

`$ sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin`
> 如果提示接受 GPG 密钥，请验证指纹是否匹配 060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35，如果是，则接受它。
> 此命令会安装 Docker，但不会启动 Docker。它还会创建一个 docker组，但是默认情况下它不会将任何用户添加到该组中。

<a name="cmU1d"></a>
## 启动并验证已安装Docker

1. 启动Docker
```bash
[root@base ~]#  sudo systemctl start docker
```

2. 通过hello-world运行映像来验证 Docker 引擎是否已正确安装
```bash
  [root@base ~]# sudo docker run hello-world

Hello from Docker!
```
> 此命令下载测试映像并在容器中运行它。当容器运行时，会打印一条消息并退出。

<a name="KEnz9"></a>
## 以非 root 用户身份管理 Docker
> Docker 守护进程绑定的是Unix Socket(套接字)而不是TCP 端口,默认情况下Unix Socket归属`root`用户，其它用户只可以通过`sudo`命令访问，Docker守护进程始终以`root`用户身份运行
> 如果不想在docker命令前加上`sudo`，可以创建一个名为docker的Unix组，并将用户加入其中。当Docker守护程序启动时，它会创建一个Unix套接字，供docker组的成员访问

<a name="rDeXg"></a>
#### 创建Docker用户组并添加用户

1. 创建`docker`组

`$  sudo groupadd docker`

2. 添加用户到`docker`组

`$ sudo usermod -aG docker $USER`

3. 激活并重新登陆

Linux系统中可通过如下命令更新对组的修改<br />`$ newgrp docker`

4. 验证无`sudo`执行docker命令

 `$ docker run hello-world`

5. 可能出现的错误

如果在将用户添加到docker组之前，使用`sudo`运行Docker CLI命令可能会出现如下错误
```bash
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```
解决思路：

   1. 删除~/.docker/目录（它会自动重新创建，但任何自定义设置都会丢失）
   1. 使用命令改变其所有权和权限
```bash
$ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
$ sudo chmod g+rwx "$HOME/.docker" -R
```
<a name="n2agu"></a>
## Docker自启动

1. 配置Docker开机自启动
```bash
 $ sudo systemctl enable docker.service

Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.

 $ sudo systemctl enable containerd.service

Created symlink from /etc/systemd/system/multi-user.target.wants/containerd.service to /usr/lib/systemd/system/containerd.service.
```

2. 关闭Docker开机自启动配置
```bash
 $ sudo systemctl disable docker.service
 $ sudo systemctl disable containerd.service
```
> 如果需要添加 HTTP 代理为 Docker 运行时文件设置不同的目录或分区，或进行其他自定义操作，参考[自定义 systemd Docker 守护程序选项](https://docs.docker.com/config/daemon/systemd/)

<a name="A23iH"></a>
## 卸载Docker

1. 卸载 Docker Engine、CLI、Containerd 和 Docker Compose 软件包：

`$ sudo yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin `

2. 主机上的映像、容器、卷或自定义配置文件不会自动删除。要删除所有映像、容器和卷(必须手动删除任何已编辑的配置文件)：

`$ sudo rm -rf /var/lib/docker $ sudo rm -rf /var/lib/containerd `
