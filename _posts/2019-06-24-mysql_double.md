---
layout: default
title: mysql使用double的坑
#excerpt: 

---

　　碰到一个很奇葩的问题，mysql默认是四舍五入的，但是，使用double，有时候不四舍五入。  

# 犯罪现场还原

## 表结构

```mysql
CREATE TABLE `test` (
  `salary_decimal` decimal(11,2) DEFAULT NULL,
  `salary_double` double(11,2) DEFAULT NULL
) ENGINE=InnoDB ;
```

## 表数据

　　salary_decimal 和 salary_double 分别插入 -96.845  

　　很神奇的是，保存后，decimal的是 -96.85 ，double的是-96.84  

　　然而，把double的精度从2增加到3，mysql会自动从 -96.84 变为 -96.845 差点亮瞎了我的眼睛  

## 引发的问题

　　这样引发了两个问题：

　　1. 两边数据不一致，一个decimal是-96.85 一个double是-96.84  
        　　2. sum函数，结果不准确。使用double的表，求和的时候，会出问题。sum(-96.84+ -3.15)的结果不是 -99.99 而是 -100.00  



# 结论  

  1. 针对问题1，特意找了下资料，发现double本来就是不精准的   

     附上官网的解释

     > For FLOAT, the SQL standard permits an optional specification of the precision (but not the range of the exponent) in bits following the keywordFLOAT in parentheses. MySQL also supports this optional precision specification, but the precision value is used only to determine storage size. A precision from 0 to 23 results in a 4-byte single-precision FLOAT column. A precision from 24 to 53 results in an 8-byte double-precision DOUBLEcolumn.

2. 针对问题2，有了问题1的前提，sum的结果自然也是不精准 