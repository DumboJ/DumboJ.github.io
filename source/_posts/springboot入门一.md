---
title: springboot入门案例
date: 2019-09-19 19:57:36
tags:
 -springboot
 -微服务
categories:
 -springboot
cover: static/bgpic/timg.jpg
---

maven:简化jar包的依赖关系

springboot：简化项目架构构建



#### Hello World 讲解

##### 项目构建-pom.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<!-- srpingboot 父模块 -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.8.RELEASE</version>
	</parent>

	<!-- web依赖，包含springMVC引用 -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

	<!-- 作为一个可执行jar -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>	

```

##### springBoot启动类

> ​		利用IDEA或者Eclipse构建一个基础SpringBoot项目后，在src/main/java下会生成一个启动类文件xxxApplication
>
> ```java
> @SpringBootApplication
> public class DemoApplication {
> 
> 	public static void main(String[] args) {
> 		
> 		SpringApplication.run(DemoApplication.class, args);
> 	
> 	}
> }
> ```
>
> ###### @SpringBootApplication注解
>
> Spring Boot应用标识
>
> ###### SpringApplication对象
>
> main函数作为应用入口，SpringAppliction引导类，run方法加载应用本身字节码文件，通过嵌入的Tomcat加载项目。

##### 快速入门案例

```java
@RestController
class ThisWillActuallyRun {

	@RequestMapping("/")
	String home() {
		"Hello World!"
	}

}
```

@RestController和@RequestMapping是SpringMVC的注解

###### @RestController

标注一个Controller，返回格式为Json数据，xml或标准字符串

另一个标识C层接口的注解@Controller，返回Json格式数据时需要在具体方法返回时加@ResponseBody注解

###### @RequestMapping

提供路由信息，前端请求会根据注解后的路径把前端请求映射到对应方法中处理。

##### 简单修改默认配置信息

springboot维护很多默认配置信息，可以通过src/main/resources/application.properties或者application*.yml/application*.yaml修改。

```xml
#修改嵌入Tomcat端口号
#服务器端口
server.port=8081
```

springboot的配置清单可移步官方配置清单查看

> https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html

##### 发布访问

启动springboot入口类的main方法

![](https://github.com/DumboJ/DumboJ.github.io/blob/hexo/source/static/sourcepic/hellospringboot.jpg?raw=true)

localhost:8081/访问页面，页面返回

> first springboot

#### 小结：

1. 入门项目搭建，pom.xml配置
2. 启动类及注解详解
3. 项目访问



