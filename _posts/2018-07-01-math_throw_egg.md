---
layout: default
title: 扔鸡蛋问题
#excerpt: 
---
　　假如有100层楼，总共有2个鸡蛋。需要多少次才能试探出临界点，比如，在第三层扔下去，不碎；在第四层扔下去，碎了，那第三层和第四层就是临界点。   
　　如果之前没准备过的话，大概第一个想到的就是二分法。   
# 1. 二分法   

　　首先在第50层丢第一个鸡蛋，若鸡蛋碎了，则在第一层开始往上丢鸡蛋，最坏情况是试探49+1次，为什么要从第一层开始尝试呢，因为只有2个鸡蛋；若鸡蛋没碎，则在75层丢第二次，若碎了则在(50,75）区间从下往上尝试。。。   

　　结论：二分法最坏尝试次数是50次。   

　　显然二分法不是一个好的解决方案，这里介绍第二种方法“平方根法”。   

# 2. 平方根法   

　　因为100是10的平方，我们可以把10作为一个区间去尝试，即第一层在第10层丢，不碎，则去第20层丢。。。，一直到第90层丢，还不碎的话，则在(91,100]层逐一尝试，最坏情况是9+10=19次，19次了，比二分法要前进了一半以上了。并且平方根法还可以再优化一下：以15层作为起点，步伐是10。即第一层是15，第二次是25，第三次是35,45...95。这种优化后的呢，最坏情况下，是在第9次（第95层碎），然后在(85,95)区间尝试9次，即优化后的最坏尝试次数是9+9=18次。   

　　结论：平方根法最坏情况下是19次，平方根优化法最坏情况下是18次。   

# 3.解方程法   

　　这里介绍下解方程法，算法的思想呢是假设最优解是x次，第一次扔的楼层也是x层。为什么第一层扔的楼层也是x层呢？因为如果最优解成立   

①当第一次扔的楼层是x+1层时，若碎了，则需要在[1,x]楼层里依次扔出，最坏情况是在x层碎，即尝试次数是：x+1次，和假设不符   

②当第一次扔的楼层是x-1层时，若碎了，则需要在[1,x-1]层里依次尝试，最快情况是在x-1层碎，即尝试次数是：x-1次，和假设不符   

　　综上所述，假设最优解是x次时，第一层扔的楼层也得是x层。   

开始扔鸡蛋了   

　　第一次是在x层扔鸡蛋，蛋不碎

　　第二次得在(x,100]区间内扔鸡蛋了，具体是哪一个层呢？应该是x-1层。为什么是x-1层呢？因为[1,100]区间，尝试次数是x次，在x层不碎，则问题转化成了在(x,100]层，最优解是x-1次，同理，扔鸡蛋的楼层数也得是x-1层。   

　　第三次得在x-2层扔   

　　以此类推...   

转化成方程式是： x + (x-1) + (x-2) + (x-3) + ... + 2 + 1 = 100，左边的都能理解，右边的为什么是100呢？因为楼层是100层，方程式的左边肯定小于等于100，取最坏情况那就是方程式右边是100。   

　　根据等差数列求和公式Sn=n(a1+an)/2 ，x向上取整得14，即第一次在14层扔，第二次是在 14+(14-1)=27层，第三次是39,50,60,69,77,84,90,95,99,100。当最坏情况是在99层碎了，即尝试次数是10+4=14次。   