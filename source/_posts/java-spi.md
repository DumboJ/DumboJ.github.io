---
title: java spi
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

​	[github](https://github.com/DumboJ/demos.git)-----spitest

​	创建一个Maven工程

​	目录结构：

​	![](https://raw.githubusercontent.com/DumboJ/DumboJ.github.io/hexo/source/static/sourcepic/javaspi.png)

​	接口类：src/main/java/package_name/People

```java
/**SPI接口*/
public interface People {
	public String skin();//肤色
}
```

具体实现类：

Chinese

```java
/**SPI实现类一*/
public class Chinese implements People {

	public String skin() {
		return "Chinese is yellow skin";//黄皮肤
	}
}
```

American

```java
/**SPI实现类二*/
public class American implements People {

	public String skin() {
		
		return "American is white skin";//白皮肤
	}
}

```

在src/main/resources下创建目录META-INF，并以接口全路径为名创建文件，文件内容为实现类的全路径名

```java
cn.dumboj.test.spi.American
cn.dumboj.test.spi.Chinese
```

测试：

```java
/**SPI测试类*/
public class TestSpi {
	public static void main(String[] args) {
		ServiceLoader<People> loader = ServiceLoader.load(People.class);
		Iterator<People> skins = loader.iterator();		//实现类迭代器
		while(skins.hasNext()){			//实现类遍历
			System.out.println(skins.next().skin());
		}
	}
}
```

测试结果：

```java
American is white skin
Chinese is yellow skin
```



#### 总结

从案例可以看出，在我们有更多具体实现需求时，只需要创建工程并基于对应接口实现，并在META-INF/services下做相应文件配置，将工程打包，就完成新服务的开发。