---
layout: default
title: record of noSuchMethod
#excerpt: 

---

#  故事背景

   一个用的很久的功能，突然运营人员反馈说不正常。经过排查，报错了，输出req对象的时候，hutool的错，说没有 dateUtil.parse方法

#  排查过程

   首先尝试复现问题，结果在测试环境复现了。报错信息一样的noSuchMethod，然鹅我在dubug的时候，是可以点进去的，说明找得到parse方法

   写个简单的main函数，调用它的方法，也是报错

   重新写一个req对象，req的内容是一样的，执行正常

# 结论

   问题的根本原因是jar包冲突。a项目有hutool的包 版本4.5，b项目有hutool的包 5.7，而a项目把b项目以jar的形式引入的。最开始两个项目hutool版本一致，后来b项目升级了hutool的版本，a项目没升级版本。这就是为什么好好的突然就不行了。。。