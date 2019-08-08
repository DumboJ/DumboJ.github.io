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
cover: static/bgpic/cat.jpg
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

​	静态代理直接通过编码创建代理类。

##### 2.JDK动态代理

###### 	接口：

```java
/**
 * @author dumboj
 *
 * */
public interface MobilePhone {
	void call();
}
```

###### 	被代理类：

```java
/**
 * @author dumboj
 * */
public class IOSMobilePhone implements MobilePhone {
	private String firstName;
	private String lastName;
	
	public IOSMobilePhone(String firstName, String lastName) {
		super();
		this.firstName = firstName;
		this.lastName = lastName;
	}

	@Override
	public void call() {
		System.out.println("use apple phone call the "+firstName+"·"+lastName);
	}

}
```

###### InvocationHandler--每个代理对象都会关联一个InvocationHandler

```java
/**
 * @author dumboj
 * */
public class MobilePhoneHandler<T> implements InvocationHandler {
	//持有被代理对象引用
	private T target;
	
	public MobilePhoneHandler(T target) {
		super();
		this.target = target;
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		//前置方法
		before();
		Object obj = method.invoke(target, args);
		//后置方法
		after();
		return obj;
	}

	private void after() {
		System.out.println("method after target method invok ...");
	}

	private void before() {
		System.out.println("method before target method invok ...");
	}

}
```

###### 测试类

```java
/**
 * @author dumboj
 * */
public class ProxyIOSMobileTest {
	public static void main(String[] args) {
		MobilePhone mobile = new IOSMobilePhone("LeBron", "James");
		InvocationHandler handler = new MobilePhoneHandler<MobilePhone>(mobile);
		MobilePhone mpProxy = (MobilePhone)Proxy.newProxyInstance(mobile.getClass().getClassLoader(), new Class<?>[] {MobilePhone.class}, handler);
		mpProxy.call();
	}
}
```

###### 执行结果

```java
method before target method invok ...
use apple phone call the LeBron·James
method after target method invok ...
```

Java的动态代理中，IncocationHandler和Proxy是两个重要的类或接口。	

Java API中的描述：

InvocationHandler

```
InvocationHandler is the interface implemented by the invocation handler of a proxy instance. 

Each proxy instance has an associated invocation handler. When a method is invoked on a proxy instance, the method invocation is encoded and dispatched to the invoke method of its invocation handler.
```

每一个动态代理类都必须实现InvocaitonHandler接口，每个代理类的实例都关联到一个Handler,当我们通过代理对象调用一个方法时，这个方法的调用会被转发为由InvocationHandler这个接口的invoke（）调用。

```
Object invoke(Object proxy, Method method, Object[] args) throws Throwable

proxy:　　指代我们所代理的那个真实对象
method:　　指代的是我们所要调用真实对象的某个方法的Method对象
args:　　指代的是调用真实对象某个方法时接受的参数
```



Proxy这个类的作用则是用来动态创建一个代理对象的类，关键看newProxyInstance()这个方法。

```
	public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler h)  throws IllegalArgumentException
Returns an instance of a proxy class for the specified interfaces that dispatches method invocations to the specified invocation handler.
```

这个方法的作用就是得到一个动态的代理对象，其中三个参数的含义：

###### ClassLoader loader

一个ClassLoader类加载器对象，定义由哪个类加载对生成的代理对象进行加载。

###### Class<?>[] interfaces

一个接口对象的数组，表示将要给被代理的对象提供一组什么接口，如果提供一个被代理类的接口对象，则认为代理对象和被代理对象实现了统一接口（多态），这样就能调用这组接口中的方法了。

###### InvocationHandler h

一个InvocationHandler 对象，表示被代理对象在调用方法时，会关联到哪一个InvocationHandler对象上。

###### 动态代理与静态代理区别	：

1.Proxy类的代码固定，不因业务庞大而需要是维护很多代理类。

2.可以实现AOP编程

3.解耦，web业务下可以实现数据层和业务层的分离

4.无侵入式代码扩展