---
layout: default
title: mybatis java.lang.reflect.UndeclaredThrowableException异常
#excerpt: 

---



​      mybatis 出现java.lang.reflect.UndeclaredThrowableException异常，出现这个问题，很奇怪，因为最近没人提交相关的代码，检查了类的字段和数据库字段是对应的，百思不得其解，还增加了日志，仍然没解决，而且只会发生在运营环境，本地是ok的，折腾了好久才发现，是序列落后了。表内的数据被覆盖，原先的序列不适用覆盖后的数据

