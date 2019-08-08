---
title: Java中的代理机制
date: 2019-08-08 10:13:21
tags: 
 -java
 -JDK动态代理
 -CGLIB动态代理
 -静态代理
categories:
 -动态代理
---

##### 1.静态代理

​	创建一个接口，被代理类实现该接口并实现该接口的抽象方法，代理类也实现该接口，即代理类与被代理类实现同一接口。代理类中持有被代理类的引用，即被代理类是代理类的成员变量，在代理类实现的接口方法中调用被代理类的方法。

​	其实就是代理类为被代理类预处理消息、过滤消息后执行被代理类，并能进行消息的后置处理。代理类与被代理类通常存在关联关系（持有被代理对象的引用）。代理类通过调用被代理类中方法提供服务。

​	<u>静态代理示例</u>

###### 接口：

​	

```java
public interface ProxyInterface {
    void excute();
}
```

###### 被代理类：

```java
/**
 * @author dumboj
 * 	被代理类
 * */
public class ByProxy implements ProxyInterface {

	@Override
	public void excute() {
		System.out.println("The Class which implements ProxyInterface excute... ...");
	}
}
```

###### 代理类：

```java
/***
 * @author dumboj 静态代理类---与被代理实现相同接口
 */
public class ProxyClass implements ProxyInterface {

	// 成员变量--接口
	private ProxyInterface proxy;

	// 构造方法多态赋值
	public ProxyClass() {
		proxy = new ByProxy();
	}

	@Override
	public void excute() {
		System.out.println("Do something before ByProxy Class excute");
		proxy.excute();
		System.out.println("Do something before ByProxy Class excute");
	}

}
```

###### 测试类：

```java
/**
 * @author dumboj 测试静态代理调用
 */
public class StaticProxyTest {
	public static void main(String[] args) {
		
		ProxyInterface proxy = new ProxyClass();
		proxy.excute();
		
	}
}
```

###### 执行结果：

```java
Do something before ByProxy Class excute
The Class which implements ProxyInterface excute... ...
Do something after ByProxy Class excute
```

​	由上知，静态代理很容易对单个类实现代理。缺点在于，静态代理只能为一个类服务，当需要代理的类增加，需编写大量代理类，工作量繁重。