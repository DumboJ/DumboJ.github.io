---
title: node升级
date: 2019-07-25 10:18:55
tags:
---

###### npm 升级

`npm i -g npm`  升级npm版本

也可以指定npm版本号： `npm i -g npm@6.10.1`

###### node 升级

1. 通过npm安装node的版本管理工具  n

   npm i -g n

2. `

   ```java
   Commands:
   
     n                              Output versions installed
     							   --查看已安装版本
     n latest                       Install or activate the latest node release
     							   --最新版本
     n -a x86 latest                As above but force 32 bit architecture
     n stable                       Install or activate the latest stable node release                          --安装稳定版
     n lts                          Install or activate the latest LTS node release
     n <version>                    Install node <version>
     							   --安装指定版本
     n use <version> [args ...]     Execute node <version> with [args ...]
     n bin <version>                Output bin path for <version>
     n rm <version ...>             Remove the given version(s)
     n prune                        Remove all versions except the current version
     n --latest                     Output the latest node version available
     n --stable                     Output the latest stable node version available
     n --lts                        Output the latest LTS node version available
     n ls                           Output the versions of node available
   ```

3. souce /etc/profile

   node -v 

   npm -v