---
layout: package
title: githubDNS问题
date: 2019-09-21 10:27:15
tags:
 -小疑问
category:
 -githubDNS设置
cover: static/bgpic/sum.jpg
---

github.com访问缓慢

打开C:\Windows\System32\drivers\etc路径下hosts文件

添加

```tex
192.30.253.112 github.com

192.30.253.119 gist.github.com

151.101.100.133 assets-cdn.github.com

151.101.100.133 raw.githubusercontent.com

151.101.100.133 gist.githubusercontent.com

151.101.100.133 cloud.githubusercontent.com

151.101.100.133 camo.githubusercontent.com

151.101.100.133 avatars0.githubusercontent.com

151.101.100.133 avatars1.githubusercontent.com

151.101.100.133 avatars2.githubusercontent.com

151.101.100.133 avatars3.githubusercontent.com

151.101.100.133 avatars4.githubusercontent.com

151.101.100.133 avatars5.githubusercontent.com

151.101.100.133 avatars6.githubusercontent.com

151.101.100.133 avatars7.githubusercontent.com

151.101.100.133 avatars8.githubusercontent.com 
```

cmd ：

```bash
ipconfig/flushdns
```

秒连..........

老人，手机，地铁？？？