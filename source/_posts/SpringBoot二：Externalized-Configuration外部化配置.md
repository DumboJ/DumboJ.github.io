---
title: SpringBoot二：Externalized Configuration外部化配置
date: 2019-10-04 17:22:31
tags:
 -springboot
 -微服务
categories:
 -springboot
cover: 
---

### SpringBoot外部化配置

[参阅官方文档]: https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/boot-features-external-config.html

SrpingBoot除了帮我们完成一些自动配置的同时，也允许我们使用外部化配置覆盖原有配置。以便于在不同的环境中使用相同的应用程序代码。

外部化配置的方式可以是下列几种：

​				--------properties文件

​				--------YAML文件

​				--------环境变量

​				--------命令行参数

属性值可以通过@Value直接注入，通过访问Spring的Environment抽象或者通过结构化对象@ConfigurationProperties绑定。

​	