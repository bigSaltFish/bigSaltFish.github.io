---
layout: default
title: new thread not work
#excerpt: 

---

#  异常信息

   在controller使用异步，代码是 new thread( () -> {  xxxService.doSomething(); } );

在本地是正常运行，能进入doSomething内。到测试环境，有些机器上是正常，个别机器上，异常，无法进入doSomething内，我debug了，感受到的是莫名其妙的被杀掉了线程。

# 问题分析

　　在本地实验了，只有把租户id改成特定的，才会异常。百思不得其解，听同事的建议，加了休眠时间，本地是正常了。重构到测试环境，那个别的机器上，还是不能正常得到想要的结果。

​    最终，把异步的代码，从jdk1.8的lambda的写法，改成jdk1.7的写法，问题就没再发生了。





神奇

