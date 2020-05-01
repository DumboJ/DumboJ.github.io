---
title: springboot中过滤器和拦截器
date: 2020-04-29 14:53:15
tags: springboot
conver: 
---

#### 过滤器

###### 1.定义过滤器

```java
package cn.dumboj.filter;

import javax.servlet.*;
import java.io.IOException;
import java.util.Date;

/**
 * @Description 过滤器执行耗时---接口实现
 * @return
 * @Author dumboj
 * @Date 2020/4/29 11:11
 * @Version 1.0
 */
//@Component
//@WebFilter(urlPatterns = "/*")
public class AnnotationTimeFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("filter init....");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("start filter");
        long startTime = new Date().getTime();
        chain.doFilter(request,response);
        System.out.println("过滤器耗时：" + (new Date().getTime() - startTime));
        System.out.println("过滤器执行完成");
    }
    @Override
    public void destroy() {
        System.out.println("销毁过滤器");
    }
}

```

###### 2.过滤条件设置和过滤器注入方式一：注解注入（推荐）

​	

```java
/**
配置方式一：注解方式
 *    {@link  @Component}  bean注入容器
 *    {@link  @WebFilter(urlPatterns = "/*")}
 *              urlPatterns 配置哪些请求可以进入过滤器，/*表示所有请求
 */
@Component
@WebFilter(urlPatterns = "/*")
public class AnnotationTimeFilter implements Filter {
	...	...
}
```

###### 3.过滤条件设置和过滤器注入方式二：过滤器注册

```java
package cn.dumboj.filter;

import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Filter;
import java.util.ArrayList;

/**
 * @Description 过滤器质性耗时--注册器实现
 * @return
 * @Author dumboj
 * @Date 2020/4/29 14:10
 * @Version 1.0
 */
@Configuration
public class RegistraTimeFilter {
    //筑造器注入
    @Bean
    public FilterRegistrationBean registrationBeanFilter(){
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        AnnotationTimeFilter timeFilter = new AnnotationTimeFilter();
        //设置具体过滤器实现
        filterRegistrationBean.setFilter(timeFilter);
        //设置过滤条件
        ArrayList<String> urlList = new ArrayList<>();
        urlList.add("/*");
        filterRegistrationBean.setUrlPatterns(urlList);
        return filterRegistrationBean;
    }
}

```

#### 拦截器

