---
title: Springboot 项目打包成war包和jar包
date: 2020-04-26 19:41:18
tags: springboot 打包
cover: static/bgpic/larq-TJVSLJtKC9Y-unsplash.jpg
---

###### 1.pom.xml文件配置

|         标签名称          |                             war                              |                             jar                              |
| :-----------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    <package></package>    |                    <package>war</package>                    |                    <package>jar</package>                    |
| <dependency></dependency> | <dependency>    <groupId>org.springframework.boot</groupId>    <artifactId>spring-boot-starter-tomcat</artifactId>    <scope>provided</scope></dependency> | <dependency>    <groupId>org.springframework.boot</groupId>    <artifactId>spring-boot-starter-tomcat</artifactId>    <scope>provided</scope></dependency> |
|      <build></build>      | <build> <finalName>项目名（类似于server.context-path=****）</finalName></build> | <build>         <plugins>             <plugin>                 <groupId>org.springframework.boot</groupId>                 <artifactId>spring-boot-maven-plugin</artifactId>                 <configuration>                     <mainClass>启动类名称</mainClass>                 </configuration>             </plugin>         </plugins>     </build> |

###### 2.添加启动类ServletInitializer

```java
/**
 * @param
 * @Description 添加启动类ServletInitializer
 * @return
 * @Author dumboj
 * @Date 2020/4/26 19:15
 * @Version 1.0
 */
public class ServletInitializer extends SpringBootServletInitializer {
    /**
     * Configure the application. Normally all you would need to do is to add sources
     * (e.g. config classes) because other settings have sensible defaults. You might
     * choose (for instance) to add default command line arguments, or set an active
     * Spring profile.
     *
     * @param builder a builder for the application context
     * @return the application builder
     * @see SpringApplicationBuilder
     */
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(SpringbootDevtoolsApplication.class);
    }
}
```

###### 3.打包

idea中maven下对应项目 Lifecycle中 clean    package,则可在target目录下生产相应的jar包和war包