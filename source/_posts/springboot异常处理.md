---
title: springboot异常处理
date: 2020-04-28 18:31:07
tags: Springboot 异常
cover: 
---

###### 1.默认异常处理

​	 Spring Boot对异常的处理有一套默认的机制：当应用中产生异常时，Spring Boot根据发送请求头中的`accept`是否包含`text/html`来分别返回不同的响应信息。当从浏览器地址栏中访问应用接口时，请求头中的`accept`便会包含`text/html`信息，产生异常时，Spring Boot通过`org.springframework.web.servlet.ModelAndView`对象来装载异常信息，并以HTML的格式返回；而当从客户端访问应用接口产生异常时（客户端访问时，请求头中的`accept`不包含`text/html`），Spring Boot则以JSON的格式返回异常信息。 

​	

```java
package cn.dumboj.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Description springboot默认异常信息处理
 * @return
 * @Author dumboj
 * @Date 2020/4/28 10:37
 * @Version 1.0
 */
@RestController
@RequestMapping("exception")
public class ExceptionController {
    @GetMapping("/demo/{id}")
    public void get(@PathVariable String id) {
        throw new RuntimeException("user not exist");
    }
}

```

> 附：源码过程分析，查看org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController类

 `errorHtml`和`error`方法的请求地址和方法是一样的，唯一的区别就是`errorHtml`通过`produces = {"text/html"}`判断请求头的`accpet`属性中是否包含`text/html` 

###### 2.自定义异常处理

1. 自定义异常处理类，继承RuntimeException

   ```java
   package cn.dumboj.customerexception;
   
   /**
    * @param
    * @Description 自定义异常类
    * @return
    * @Author dumboj
    * @Date 2020/4/28 16:36
    * @Version 1.0
    */
   public class ExceptionDemo extends RuntimeException{
       private String id ;
       public ExceptionDemo (String id){
           super("this is a customer Exception");
           this.id = id;
       }
   
       public String getId() {
           return id;
       }
   
       public void setId(String id) {
           this.id = id;
       }
   }
   
   ```

   

2. 定义处理自定义异常的Handler

   ```java
   package cn.dumboj.customerexception;
   
   import org.springframework.http.HttpStatus;
   import org.springframework.web.bind.annotation.ControllerAdvice;
   import org.springframework.web.bind.annotation.ExceptionHandler;
   import org.springframework.web.bind.annotation.ResponseBody;
   import org.springframework.web.bind.annotation.ResponseStatus;
   
   import java.util.HashMap;
   import java.util.Map;
   
   /**
    * @param
    * @Description 自定义异常处理类
    * @return
    * @Author dumboj
    * @Date 2020/4/28 16:41
    * @Version 1.0
    */
   @ControllerAdvice
   public class ExceptionControllerHandler {
       /*
       * @Description 处理异常
       * * @param
       * @ExceptionHandler 指定要处理的异常类型
       * @ResponseStatus   异常处理方法返回的HTTP状态码
       * HttpStatus 异常类型枚举
       * @Return
       * @Author dumboj
       * @Date 2020/4/28 16:45
       **/
   
       @ExceptionHandler(ExceptionDemo.class)
       @ResponseBody
       @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
       public Map<String ,Object> handerExceptionDemo(ExceptionDemo demo){
           Map<String,Object> map = new HashMap<>();
           map.put("id",demo.getId());
           map.put("message",demo.getMessage());
           return map;
       }
   }
   
   ```

3. 测试自定义异常的使用

   ```java
   package cn.dumboj.customerexception;
   
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   /**
    * @Description 测试自定义异常
    * @return
    * @Author dumboj
    * @Date 2020/4/28 17:06
    * @Version 1.0
    */
   @RestController()
   public class ExceptionTestController {
   
       @RequestMapping("/customer/{id}")
       public void testCustomerEx(@PathVariable String id){
           throw new ExceptionDemo(id);
       }
   }
   
   ```

   

  

