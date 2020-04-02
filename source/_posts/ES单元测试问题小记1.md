---
title: ES单元测试问题小记1
date: 2020-04-02 20:25:24
tags: ES6.3.2
---

##### 问题描述：

##### 		Window 搭建完Elasticseach 环境，使用localhost 本机都能够正常访问Elasticsearch 环境，使用Springboot +集成elasticsearch 提示如下错误信息：NoNodeAvailableException[None of the configured nodes are available

##### 1.相关代码：

######  单元测试

```java
package cn.dumboj.springbootes;

import cn.dumboj.springbootes.entity.ES.ESCoreDict;
import cn.dumboj.springbootes.repository.ES.ESCoreDictRepository;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import javax.swing.*;
import java.util.Iterator;
import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest
class SpringbootEsApplicationTests {

    @Autowired
    ESCoreDictRepository esCoreDictRepository;
	
    @Test
    public void testEs(){
        Iterable<ESCoreDict> all = esCoreDictRepository.findAll();
        Iterator<ESCoreDict> iterator = all.iterator();

        if (iterator.hasNext())
            System.out.println("-----------"+iterator.next().getName());
    }
}

```



###### ESRepository代码

```java
public interface ESCoreDictRepository extends ElasticsearchRepository<ESCoreDict,Integer> {
}

```

###### ES+JPA properties项目文件配置

```properties
#JPA配置相关
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect

#ES
spring.data.elasticsearch.cluster-nodes=127.0.0.1:9300
spring.data.elasticsearch.cluster-name=my-application
```



##### 2.单元测试

​	ES6.3.2  windows X64  本地环境启动并 运行单元测试。

######     Exception message:配置文件中没有可以使用的节点

![]( [https://raw.githubusercontent.com/DumboJ/DumboJ.github.io/hexo/source/static/sourcepic/ES%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%E9%94%99%E8%AF%AF%E5%B0%8F%E8%AE%B01.png](https://raw.githubusercontent.com/DumboJ/DumboJ.github.io/hexo/source/static/sourcepic/ES单元测试错误小记1.png) )



##### 3.解决方式



打开并修改本地ES安装目录下/config/elasticsearch.yml文件参数（默认注释）



| 参数          | 配置           |
| ------------- | -------------- |
|               |                |
| cluster.name: | my-application |
| network.host: | 127.0.0.1      |
| http.port:    | 9200           |



##### 4.重启ES服务，单元测试启动成功。