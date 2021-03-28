---
layout: default
title: java里为什么要set/get方法？
#excerpt: 

---

　　今天在刷掘金的时候，看到一篇文章 [Java Getter/Setter “防坑指南” 来了](https://juejin.im/post/5d9b25865188250969774e64) 里面有说道：

> 通过使用 Getter/Setter 方法，变量的访问(get)和更新(set)将变得可控。考虑以下 Setter 方法的代码

```java
public void setName(String name) {
    if (name == null || "".equals(name)) {
        throw new IllegalArgumentException();
    }
    this.name = name;
}
```
　　感觉有点不对劲啊，因为之前的公司，都是禁止在set/get方法里添加任何逻辑的，实体类就是单纯的实体类，不做任何的逻辑处理，任何。   

　　既然文章的论据不充分，那么java里到底是为何需要set/get方法呢？而且还已经成为一种javaer的常识了  
　　网上有好几个说法，有的说是为了体现java语言的封装特性、有的说是为了安全，有的说是为了符合规范，都说的不是很到位。  
　　其实，使用set/get就是为了对变量的控制  
　　假如有一个属性，可以提供给外部查看，但是禁止修改，这样只需要提供get()，不提供set()即可。如果使用对象.属性的方式，是无法做到的。另外，spring框架里，实体类都是有生成set/get的，所以这也形成了一种规范了