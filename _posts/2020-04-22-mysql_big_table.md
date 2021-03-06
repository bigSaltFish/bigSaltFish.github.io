---
layout: default
title: MySQL大表数据处理
#excerpt: 

---

　　最近，领导发话了，要处理数据量很大的表，提前规避因数据量大而导致，查询、更新操作缓慢的问题。

　　数据量很大的表，我看了下，最多的records表，2400w条记录。

　　拉上DBA，一起断断续续的讨论了一个多星期，确定了方案(还不是最终方案，哎，跨部门协作，效率是真低)

　　前前后后讨论了四种方案，这里记录一下心路历程。



# 1.分区 - 时间

　　最初是DBA提出的分区方案，分区键是时间。然后，我提供了时间字段，就坐等完工了。心里还美滋滋的想，哇，这次的任务好轻松啊！哪知道，是我太年轻了。

　　临上线之前，DBA把所有的东西都找我确认一下。然后，我发现一个很奇怪的事情，他把我提供的时间，加到 唯一键里了。

　　WTF！ 本来是三个字段来约束唯一的，你加上 时间，那不就违背了之前的逻辑了吗？果断否了！

　　然后，DBA解释道：这是MySQL语法规定，若分区键不在唯一键里，会无法建立分区键。

　　我听了，好像挺有道理的哈。但是，违背了原逻辑，还是得否啊。

　　这里插两句

　　　　1.mysql中，一个表，只允许拥有一个唯一键。所以，搞两个唯一键，一个保持原逻辑，一个加上时间，这样是不行的。

　　　　2.分区后，程序里的sql需要带上 分区键，也就是那个时间。不然，mysql不知道你应该查哪个分区，sql会在所有的分区都执行一遍，多次IO。



# 2.分区 - 数据量

　　前面说的，按时间维度来分区，行不通。我想了一下，能否根据数据量来分，比如，1-500w记录，在分区一；500w-1000w记录，在分区二。

　　结果，DBA给我否了。理由是，他们不好管理维护。。。

　　不好维护？我去，哪里不好维护了？再问人家，他居然不鸟我了。。。

　　后来我查了下，假如用id作为分区键，用户表，2000w数据，4个分区。如果分区一，也就是用户id 1-500w的，非常活跃，查询大部分都是查他们。那么会造成逻辑上的数据不均衡，失去了分区的意义

　　按照时间维度，数据相对会更加均衡。并且若想按照时间进行数据归档，则只需要对某一个分区数据进行归档即可。所以，网上基本上也是推荐根据时间去分区的。



# 3.分表方案

　　这是DBA脑抽想出来的。

　　DBA希望有几个月份，就有几张表。现状是一张表records，按照他的想法，会有4张表，records_202001、records_202002、records_202003、records_202004。这不是扯淡吗，那代码里得改多少地方啊，果断否了



# 4.数据定时迁移

　　最终，DBA提议数据迁移：每个月或者每个季度，把一年前的数据，迁移到备份库，然后把原表的数据删掉。

　　这样的话，代码不需要改动，如果业务一定要看一年之前的数据，那我们只需要再部署一套代码，把数据源配置改为备份库即可。



　　定时迁移方案，会出什么问题呢？目测没问题，不过，这一切只有等真正实施后才会知道。



# 5.大结局

　　这里的内容更新于2020年4月30日

　　在24号的时候，就已经变更了方案。领导们抉择了方案1  分区-时间。那分区键得出现在唯一索引里，这个是怎么破的呢？

　　DBA领导想了一出方案，让在加个竖表，它只存唯一索引的几个字段。因为破坏了records表的唯一键，所以，插入records的时候，先插入竖表，竖表遵循了records的唯一限制，竖表插入成功，则表示数据维持了records原逻辑下的唯一；再插入records表。

　　其实这里的思路，跟“一个表搞两个唯一索引”是一样的，只不过这样的方式更加巧妙，绕开了MySQL唯一索引数量限制。