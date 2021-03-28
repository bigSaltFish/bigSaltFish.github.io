---
layout: default
title: kafka Magic v1 does not support record headers
#excerpt: 

---

#  异常信息

```java
java.lang.IllegalArgumentException: Magic v1 does not support record headers
```

# 问题分析

　　项目中使用了kafka，作为系统间信息传递桥梁。奇怪的是，代码已经使用了一个多月的，为什么突然会出现问题？捋了捋，最近有合并分支的行为。在合并分支之前，是正常工作的，合并分支后，就出现了异常。　　

　　新旧分支的区别：

　　　　旧：SpringBoot+kafka

　　　　新：SpringCloud+kafka

　　试着切回旧版本，一切正常