---
layout: default
title: 内连接、左连接 是笛卡尔积？
#excerpt: 
---
　　这周的部门周会，分享的同事说的是数据库优化相关，过程中，一个同事跟我讨论左连接查询，是不是笛卡尔积。我第一反应，左连接肯定不是笛卡尔积啊，左连接是以左表为准，左表有m条记录，则结果集是m条记录(哈哈，如果是你，你是不是也是这样的反映)，同事听了，说内连接会是笛卡尔积。听到这句话的我的表情是这样的    

![黑人问号]({{site.url}}/assets/common/wenhao.jpg)    

　　散会后，在数据库里试验了一下，发现，事实比想象中要复杂。首先说下结论：链接查询，如果on条件是非唯一字段，会出现笛卡尔积(局部笛卡尔积)；如果on条件是表的唯一字段，则不会出现笛卡尔积。    

　　下面是具体的试验：    

　　文中会有两张表，user表和job表，表数据如下    

![user data]({{site.url}}/assets/2018-07-14-innerJoin_leftJoin/user_data.jpg)

![job data]({{site.url}}/assets/2018-07-14-innerJoin_leftJoin/job_data.jpg)

# 交叉连接    

　　

    SELECT
         *
    FROM
         `user` CROSS JOIN job;  


这种等同于(交叉查询等于不加on的内连接)
    
    SELECT
        *
    FROM
        `user` , job;



　　sql执行结果：    

![crossJoin]({{site.url}}/assets/2018-07-14-innerJoin_leftJoin/crossJoin.jpg)    

　　结论：交叉连接，会产生笛卡尔积。    

# 内连接

## 内连接唯一字段

```sql
SELECT
    *
FROM
    `user` u JOIN job j ON u.JOB_ID=j.ID;
```

![innerJoinUnique]({{site.url}}/assets/2018-07-14-innerJoin_leftJoin/innerJoinUnique.jpg)

　　结论：假如，内连接查询，on条件是A表或者B表的唯一字段，则结果集是两表的交集，不是笛卡尔积。    

## 内连接非唯一字段

 　　如果A表有m条记录，m1条符合on条件，B表有n条记录，有n1条符合on条件，则结果集是m1*n1

```sql
SELECT
    *
FROM
    `user` u JOIN job j ON u.valid=j.valid;
```

![innerJoinNotUnique]({{site.url}}/assets/2018-07-14-innerJoin_leftJoin/innerJoinNotUnique.jpg)

　　结论：假如，on条件是表中非唯一字段，则结果集是两表匹配到的结果集的笛卡尔积(局部笛卡尔积)    。

# 外连接
## 左连接

### 左连接唯一字段

　　假如A表有m条记录，B表有n条记录，则结果集是m条    
　　
      SELECT
            *
        FROM
            `user` u LEFT JOIN job j ON u.JOB_ID=j.id;    

![leftJoinUnique]({{site.url}}/assets/2018-07-14-innerJoin_leftJoin/leftJoinUnique.jpg)

结论：on条件是唯一字段，则结果集是左表记录的数量。

### 左连接非唯一字段

　　如果A表有m条记录，m1条符合on条件，B表有n条记录，有n1条符合on条件，则结果集是 (m-m1) + m1*n1

    SELECT
        *
    FROM
        `user` u LEFT JOIN job j ON u.VALID=j.VALID;
![leftJoinNotUnique]({{site.url}}/assets/2018-07-14-innerJoin_leftJoin/leftJoinNotUnique.jpg)

　　结论：左连接非唯一字段，是局部笛卡尔积。    

## 右连接

　　同左连接，这里就不赘述了

## 全外连接

　　mysql不支持