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

---------------------------

修改后几日发现上传文件时，在markdown文件中添加图片路径需要访问该图片地址，访问时一直被浏览器拦截，github全站图片信息都不加载。将上述配置注释，刷新DNS设置，访问正常。

++++++++++NDS相关知识