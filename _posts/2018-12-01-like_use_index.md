---
layout: default
title: like百分号前置不会走索引？NO！
#excerpt: 

---

　　“模糊查询，前置百分号不走索引；后置百分号才会走索引"这可能是大部分人都知道的“常识”，然而，这周在做SQL优化的时候，无意中碰到了意外情况--模糊查询，前置百分号也走索引！

举个栗子
表： TEST_USER
索引：INDEX_MOBILE

```sql
CREATE TABLE `TEST_USER` (
  `ID` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `NAME` varchar(64) DEFAULT NULL COMMENT '名字',
  `MOBILE` varchar(11) DEFAULT NULL COMMENT '手机号',
  PRIMARY KEY (`ID`),
  KEY `INDEX_MOBILE` (`MOBILE`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='测试的用户表';
```



# 测试1，只查索引字段

　　只查手机号

![只select手机号]({{site.url}}/assets/2018-12-01-like_use_index/1.jpg)

　　从上面图片中的执行计划可以看出，查询是走了索引的



# 测试2，查索引字段和主键

　　查手机号和主键

![select手机号和主键]({{site.url}}/assets/2018-12-01-like_use_index/2.jpg)

　　从执行计划中看，也是走了索引的



# 测试3，查非索引字段

　　select 索引字段和非索引字段

![select手机号和姓名]({{site.url}}/assets/2018-12-01-like_use_index/3.jpg)

　　从执行计划，可以看出这次是没有走索引的了



# 测试4，where后面多条件

　　假如where后面，多个条件

![where后面多个条件]({{site.url}}/assets/2018-12-01-like_use_index/4.jpg)

　　还是会走索引了！

# 结论

　　从上面几次试验，可以得到一个结论：like查询百分号前置，并不是100%不会走索引。如果只select索引字段，或者select索引字段和主键，也会走索引的。

　　当然，文章说的这种情况，是比较偏的，实际工作，很少只select索引字段的，但是，知道这个，以后跟别人讨论到“like百分号前置不走索引”的时候，你的内心可以是这样了

![退后 危险]({{site.url}}/assets/common/back_danger.png)



Tips：

　　文中使用的是mysq数据库，版本5.7，没有对其他版本mysql进行验证，如果其他版本不适用，请留言指正！