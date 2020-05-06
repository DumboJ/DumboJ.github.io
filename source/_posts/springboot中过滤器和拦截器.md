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

Springboot中定义拦截器，

###### 条件一：

实现org.springframework.web.servlet.HandlerInterceptor接口

```java
package cn.dumboj.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Date;

/**
 * @Description 自定义拦截器
 * @return
 * @Author dumboj
 * @Date 2020/4/29 19:15
 * @Version 1.0
 */
@Component
public class CustomerInterceptor implements HandlerInterceptor  {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截预处理：");
        request.setAttribute("startTime",new Date().getTime());
        String beanName = (((HandlerMethod) handler)).getBean().getClass().getName();
        String methodName = (((HandlerMethod) handler)).getMethod().getName();
        System.out.println("类方法：" + beanName + "." + methodName);
        return true;
    }


    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("开始处理拦截");
        Long  startTime = (Long) request.getAttribute("startTime");
        System.out.println("拦截器postHandle()执行完耗时："+(new Date().getTime()-startTime));
    }


    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("处理拦截后");
        Long  startTime = (Long) request.getAttribute("startTime");
        System.out.println("拦截器 afterCompletion ()执行完耗时："+(new Date().getTime()-startTime));
        System.out.println("异常信息:"+ex);
    }
}
```

HandlerInterceptor接口中三个方法实现：

- ​	preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)

  ​		*处理拦截之前执行*

- postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)

  ​		*被拦截的方法没有抛出异常时才执行*

- afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)

  ​		*无论被拦截方法是否抛出异常都会执行*

###### 条件二：

1. 自定义拦截器类上加入@Component注解；

2. 添加配置类,实现WebMvcConfigurer接口，并重写addInterceptors（），注册自定义过滤器

   ```java
   package cn.dumboj.interceptor;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
   import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
   import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
   import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
   
   import javax.annotation.Resource;
   
   /**
    * @Description 注册拦截器
    * @return
    * @Author dumboj
    * @Date 2020/4/30 16:14
    * @Version 1.0
    */
   @Configuration
   public class InterceptConfig implements WebMvcConfigurer {
       /**
        * Add Spring MVC lifecycle interceptors for pre- and post-processing of
        * controller method invocations and resource handler requests.
        * Interceptors can be registered to apply to all requests or be limited
        * to a subset of URL patterns.
        *
        * {@link WebMvcConfigurerAdapter}
        *  as of 5.0 {@link WebMvcConfigurer} has default methods (made
        *  * possible by a Java 8 baseline)
        * @param registry
        */
   
       @Resource
       private  CustomerInterceptor customerInterceptor;
   
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           registry.addInterceptor(customerInterceptor);
       }
   }
   ```

   ​	