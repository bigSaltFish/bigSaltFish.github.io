---
layout: default
title: eclipse自动补全功能的缺陷
#excerpt: 
---
　　这周在做一些代码的重构，在重构的过程中，踩了一个eclipse的大坑！这里记录一下    

　　在eclipse里写java代码，你写好方法名字，再来个回车，eclipse会自动帮你把方法的参数给补全，补全的参数跟你在补全方法定义的参数名字“一致”，即我认为的效果是这样的   

```java
public void test01(Integer id,String name,Integer age){
    test02(id,name,age)
}
public void test02(Integer id,String name,Integer age){	
}
```

　　一般情况下，使用起来也是这种效果的。但是，eclipse的方法参数补全，是有缺陷的

```java
public void test01(Integer id,String name,Integer age){
	Integer auctionId = 1001;
	test02(auctionId, name, age);
	
	Integer countNumber = 1001;
	test02(auctionId, name, age);
	
	int userId = 1;
	test02(auctionId, name, age);
}

public void test02(Integer id,String name,Integer age){
	
}
```
　　这test01方法，第三行，我调test02，理想中情况，是应该给我补全参数是 id和name、age，实际呢，给我补全的是auctionId！

　　然后我又试了其他的情况，发现，eclipse的补全规则，是模糊参数匹配，如果有多个参数符合条件，则就近定义原则，在调用方法距离最近的并且参数类型匹配的，权值最高！    

　　想不通，为什么eclipse不是参数名精确匹配呢？就这个不起眼的问题，给我造成了好几个bug，还不好排查，只能肉眼去一个一个扫雷。我又试了idea，但是，idea是直接没补全参数的这个功能。。。