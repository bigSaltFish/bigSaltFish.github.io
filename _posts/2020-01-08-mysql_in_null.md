---
layout: default
title: MySQL中IN对NULL的处理
#excerpt: 

---

　　先说结论，MySQL中NULL是不参与计算的，对NULL进行计算，只可以使用预设的IS NULL和IS NOT NULL来操作

　　student表中，有三条数据，id分别是1、2、3

```mysql
SELECT
    *
FROM
    student
WHERE
    id IN (1,2,NULL)
```

　　会查询出来id=1和id=2的记录


```mysql
SELECT
    *
FROM
    student
WHERE
    id NOT IN (1,2,NULL)
```


　　查询结果是空，上面的sql可以理解为

```mysql
SELECT
    *
FROM
    student
WHERE
    id !=1 AND id !=2 AND id != NULL
```

　　而上面有说到结论，MySQL的NULL是不能直接参与计算的，若违背规则，会导致最终查询的结果集是空

