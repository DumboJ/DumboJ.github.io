---
title: 'java spi '
date: 2019-09-25 21:47:23
tags: 
 -java基础
 -java spi
category:
 -java spi
 -java 基础
---

#### Java SPI简介

java SPI ：java Service Provider Interface 

​	是java中的一个服务加载方式。根据Java SPI规范，可以定义一个接口，具体实现由服务提供者去具体实现。在使用时可以通过SPI的规范获取对应实现类。通过SPI服务加载机制可以实现服务的注册和调用，有效避免代码耦合，基于接口编程，实现模块间的解耦。

#### SPI应用场景

1. JDBC中数据的驱动加载
2. dubbo框架中也用到SPI原理，dubbo对JAVA SPI进行封装，可以通过过滤器加载部分实现类

#### SPI规范

1. 在项目main/resource/META-INF/services下创建以接口全路径名为包名的文件，文件内容为具体需要加载实现类的全包名路径
2. 使用ServiceLoader动态加载实现类：具体为ServiceLoader加载接口字节码对象，通过对ServiceLoader对象Iterator迭代获取所有实现类
3. SPI实现打成jar包，需要放在classpath中
4. 实现类必须有无参构造。（有带参构造时需显式声明）

#### SPI简单案例