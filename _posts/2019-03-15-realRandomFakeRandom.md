---
layout: default
title: 真随机数和伪随机数
#excerpt: 
---

# 随机数的特性

　　随机数最初是应用在密码学的，后来也逐渐大量使用于编程语言领域。关于随机数，它有三个特性：  

1. 随机性：完全乱序；
2. 不可推测性：从已有的数，无法推测出下一个数；
3. 不可重复性：随机数之间不重复。

# 真随机数和伪随机数

## 真随机数

　　真随机数是伴随着物理实验的，比如：抛硬币、掷骰子、电子元件的噪音、核裂变等，它的结果符合三大特性的。  

　　具体实现：intel通过电阻和振荡器生成热噪声作为信息熵资源；Unix/Linux的 /dev/random 和 /dev/urandom采用硬件噪音生成随机数  

　　优点：真实随机数  

　　缺点：需要硬件配合，技术要求高，效率低  

## 伪随机数

　　伪随机数，是通过一定算法，获得一个随机的值，并不是真的随机。伪随机又分为 【强伪随机数】 和 【弱伪随机数】  

### 强伪随机数

　　强伪随机数：更加贴近【真随机数】，满足特性的：随机性和不可推测性，难以预测  

　　具体实现：java的SecureRandom随机数生成器，就是强伪随机数，因为它内部是使用 击键动作 来作为种子，而击打键盘操作是物理操作，且是不规律的。

### 弱伪随机数

　　弱伪随机数：满足随机性，可以预测

　　具体实现：典型的比如java语言里的Random生成器，它是使用时间作为种子(线索)去构造生成器的，假如攻击者获得了构造生成器的时间，那么就可以预测到下一个随机数。  



# 总结

  　　1. 安全系数高、随机性要求高，推荐使用SecureRandom；  
    　　2. 要求不高，使用Random即可；  
      　　3. 说到Random随机数，Collections类下面有一个随机排序算法--shuffle洗牌算法，其内部也是借助random来实现的。  



参考文档：  

[Random vs Secure Random numbers in Java](https://www.geeksforgeeks.org/random-vs-secure-random-numbers-java/)  

[Fisher–Yates shuffle 洗牌算法](https://gaohaoyang.github.io/2016/10/16/shuffle-algorithm/)