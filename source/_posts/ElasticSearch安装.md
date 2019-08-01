---
title: ElasticSearch安装
date: 2019-08-01 17:13:50
tags:
 -elasticsearch环境安装
categories:
 -elasticsearch
cover: /static/bgpic/xx.jpg
---

##### Elasticsearch安装

安装准备：

| 环境               | 版本     | 说明                      | link                                                         |
| :----------------- | -------- | ------------------------- | ------------------------------------------------------------ |
| windows            | 7        |                           |                                                              |
| elasticsrarch      | 7.3.0    |                           | https://www.elastic.co/cn/downloads/elasticsearch            |
| elasticsearch-head |          | git仓库master，可视化界面 | ssh:git@github.com:mobz/elasticsearch-head.git<br />https:https://github.com/mobz/elasticsearch-head.git<br />Chrome插件 |
| nodejs             | v10.16.0 | es-head运行环境           | http://nodejs.cn/download                                    |
| JDK                | 1.8      | es运行环境                |                                                              |

1.elasticsearch解压

bin目录下`elasticsearch.bat`可执行文件启动es服务	

2.`elasticsearch-head`目录下打开cmd窗口

​	执行以下命令



```
npm install -g grunt-cli
npm install
grunt server
	Started connect web server on http://localhost:9100 --可视化服务启动
```

3.localhost:9100访问可视化界面

http://localhost:9200/服务连接不上，修改ES安装目录下

/config/elasticsearch.yml

添加：

```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

重启es服务，访问可视化界面，成功连接链接http://localhost:9200/