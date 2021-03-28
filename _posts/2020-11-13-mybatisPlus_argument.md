---
layout: default
title: mybatis plus argument type mismatch
#excerpt: 

---

#  异常信息

　　项目中使用了mybatis-plus，一个功能，昨天还使用的好好的，今天突然就不能用了，控制台还报错 argument type mismatch 

```java
java.lang.illegalArgumentException：argument type mismatchorg.mybatis.spring.MyBatisSystemException：netsted exception is org.apache.ibatis.reflection.ReflectionException：Error instantiating class com.xxx.WithdrawEntity with invalid type(Builder) or values(3) , Cause：java.lang.IllegalArgumentException：argument type mismatch
```



# 问题分析

　　其中 WithdrawEntity 是我们自己定义的实体类，Builder 就不知道是什么鬼了

　　然后，去谷歌一下，发现说的都是 字段类型和实际类型不匹配，比如java.util.Date 错误引了java.sql.Date；或者是没加字段注解什么的。这些我都检查了下，都不符合，并且昨天还是好好的，怎么今天就不ok了？

　　想了想，感觉报错信息里的 Builder 很奇怪啊，这个难道是谁加上去的吗？查了下git提交记录，发现，真的是有人提交上去的，不仅提交上去了，还改了实体类的构造方法！

　　旧代码：
　　
```java
public class WithdrawEntity{
   private Integer id;
   private String name;
   ......
   set/get
}
```



　　新代码：
　　
```java
public class WithdrawEntity{
   private Integer id;
   private String name;
   ......
   set/get
   
   public WithdrawEntity(Builder builder){
     this.id = builder.id;
     this.name = builder.name;
   }
   
   public static class Builder{
     private Integer id;
     private String name;
     ......
     set/get
   }
}
```



　　这么一对比，就明了了。新代码设置了构造方法，mybatis-plus在查询出数据，组装WithdrawEntity的时候，抛异常了。因为WithdrawEntity只有一个构造方法，并且需要提供Builder参数，mybatis-plus无法提供

builder，最终导致实例化失败。

# 　　问题总结：

## 　　1. 碰到异常，不要惯性的直接去谷歌

　　其实mybatis-plus的报错信息已经说的非常明白了
　　

	Error instantiating class com.xxx.WithdrawEntity with invalid type(Builder) 

　　然而，我还是惯性思维，碰到报错就谷歌，ε=(´ο｀*)))唉 得改啊

## 　　2.实体类里不要做任何逻辑操作
　　重申再重申，实体类里不要做任何逻辑操作