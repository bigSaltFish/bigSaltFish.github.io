---
layout: default
title: MyBatis拦截参数 自动带上limit
#excerpt: 

---

　　现在公司项目里都是使用mybatis，突然有一天，看到一个奇怪的现象，在controller、service和dao都没有设置查询的 limit x,y ，但是，放开sql打印，确确实实打印了limit语句，并且实际效果也是limit的。  

　　吓得我关了IDE，清了缓存，再试，居然还是一样的。  

　　奇了怪了，这个为什么会自动加上limit查询呢？难道是mybatis新出的黑科技？但是，我看mybatis的版本，也才mybatis-spring-boot-starter 1.3.2，还是2018年3月份的，如果那个时候就出了，肯定网上资料一大堆啊，可我搜了关键字，也没看到黑科技的信息。  

　　没办法，只能本地下断点，一点点F6，终于，发现了目标 PagePlugin 对象，PagePlugin实现了 mybatis的interceptor，点进去一看，原来，在执行select语句之前，会通过ParameterHandler进行参数拦截，自动给你加上limit 和算好的起始位置、size，还顺带把 count值计算了。  

　　看来自己的知识面，还是不够广泛啊。。