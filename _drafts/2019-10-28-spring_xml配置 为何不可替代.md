---
layout: default
title: Spring xml配置 为何不可替代
#excerpt: 

---

　　在看一篇文章，关于Spring注解，文章中说道：xml配置、注解配置、java配置，三足鼎立，xml具有不可代替性。我感觉有点奇怪，现在项目都是Spring boot，已经流行java配置的方式了，古老的xml配置为什么还不可替代呢？
　　先说结论：XML不可代替，因为加载顺序。

　　

　　从加载顺序来看，jdk jar > 第三方jar > 项目java代码，第三方jar肯定是优先于项目java代码的。

　　比如要使用Mybatis，我们需要配置dataSource、sessionFactory、mapperScanner这几个bean
是可以通过java配置的方式得到dataSource和sessionFactory，但是，如何把java配置的sessionFactory交给Mybits的mapperScanner呢？
　　假如mapperScanner要接收java配置的sessionFactory，那么mapperScanner必须可以感知到我们项目里的java代码。显然，这是不安全的，存在安全隐患，sessionFactory代码和Mybits的类，没有父子类的关系，Mybatis可能需要把所有代码都扫描一遍。

那为什么xml配置的方式就是可以的呢？
　　Mybatis官方提供了一个mybatis-spring-x.jar，包中实现了根据xml配置生成指定类，再通过自定义处理，把Spring容器和xml配置bean关联起来

