---
layout: default
title: mybatis插入数据  获取自增id异常
#excerpt: 

---

　　错误信息：

Error getting generated key or setting result to parameter object. Cause:org.apache.ibatis.binding.bingingException: parameter 'id' not found. Available parameters are ...



  出现问题原因，在DAO中的入参，是字符串，而不是对象，导致 插入后，返回值无法塞值到 对象中