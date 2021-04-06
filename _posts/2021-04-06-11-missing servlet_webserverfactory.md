---
layout: default
title: Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean
#excerpt: 

---

#  异常信息

```java
Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean
```

# 问题分析

　　按照网上的资料，@SpringBootApplication注解是存在的、spring-boot-starter-web的依赖也是存在的，百思不得其解，然后问了其他同事，他们本地是启动正常的。

　　又把spring-boot-starter-tomcat的scope注释了，仍然不行。

```java
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-tomcat</artifactId>
     <!--<scope>provided</scope>-->
</dependency>
```

　　折腾了好一会，才解决，原来，是我的机器是新的，idea的配置没设置。我的idea版本是2020.2，需要设置 run/debug configurations 中application 勾选 include dependencies with provided scope(该选项默认是不勾选)

!({{site.url}}/assets/2021-04-06-11-missing servlet_webserverfactory/idea_run_configurations.png)

