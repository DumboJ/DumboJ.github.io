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

[官方配置介绍文档](https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/boot-features-external-config.html )

[官方配置文档](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)

### 外部化配置参数方式

SrpingBoot除了帮我们完成一些自动配置的同时，也允许我们使用外部化配置覆盖原有配置。以便于在不同的环境中使用相同的应用程序代码。

外部化配置的方式可以是下列几种：

​				--------properties文件

​				--------YAML文件

​				--------环境变量

​				--------命令行参数

属性值可以通过@Value直接注入，通过访问Spring的Environment抽象或者通过结构化对象@ConfigurationProperties绑定。在与@ConfigurationProperties映射时，prefix=“param”,param不能以大写字母开头，必须是小写,有分隔符必须是连字符-，比如my-name，my_name不行。否则项目启动时报错：	

```java
Description:

Configuration property name 'Student2' is not valid:

    Invalid characters: 'S'
    Bean: /stu2
    Reason: Canonical names should be kebab-case ('-' separated), lowercase alpha-numeric characters and must start with a letter

Action:

Modify 'Student2' so that it conforms to the canonical names requirements.
```



​	

```java
    SpringBoot Configration Processor not found int classpath //错误信息	
```

​	原因:spring配置注解未在classpath中未找到，springboot1.5以上@ConfigurationProperties取消位置注解

​	解决方案：在Entity实体上加注解@PropertySource("application.properties")

### 配置随机数

随机数属性源（RandomValuePropertySource）可用于注入随机数。它可以生成int,long,uuid,String等数据类型的随机数，例如

```java
my.secret=${random.vaue}
my.number=${random.int}
my.uuid=${random.uuid}
my.bignumber=${random.long}
my.number.less.than-ten=${random.int(10)}
#某个范围内的值
my.number.in-range=${random.int[15,999]}
```

### YAML配置和properties配置

> properties配置就是普通的键值对 key=value 形式。

例：

```java
website.github=www.github.com
```

> YAML配置：YAML是JSON的超集，是一种用于指定层次结构配置数据的便捷格式。只要在类路径上具有SnakeYAML库，SpringApplication类就会自动支持YAML作为属性的替代方法。

#### yaml文件相关说明

​		spring提供两个对应的工厂类，可用于加载YAML文件。`YamlPropertiesFactoryBean` 	加载属性作为properties键值对，`YamlMapFactoryBean` 加载属性作为Map对象。

例如配置文件application.yaml

```yaml
environment： 
	#生产环境
	prod:
		url: http://dumboj.top
		name: myblog
	#开发环境
	dev:
		url: http://dumboj.github.io
		name: myblogDev
##!!!YAML的相同缩进代表同级，带属性值的key：value,:后必须有空格
```

示例将被转换成以下属性：

```properties
environment.prod.url=http://dumboj.top
environment.prod.name=myblog
environment.dev.url=http://dumboj.github.io
environment.dev.name=myblogDev
```

##### YAML文件可以用[index]表示属性值。

例如：

```yaml
server:
	url:
		-localhost: 8080
		-localhost: 8081
```

可以转化成：

```properties
server.url[0]=localhost:8080
server.url[1]=localhost:8081
```

##### `@ConfigurationProperties` 绑定属性

提供对应属性的setter方法，上面例子就可以表示为：

```java
@ConfigurationProperties(prefix="server")
public class config(){
	private List<String> url = new ArrayList<String>();	
	public List<String> getUrl() {
		return this.Url;
	}
}
```

