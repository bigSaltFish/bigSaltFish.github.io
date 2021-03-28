---
layout: default
title: spring boot输出日志到文件，遇到的问题
#excerpt: 
---



　　一个非常简单的问题，我却纠结了半个下午。
　　Spring boot的日志默认是输出控制台的，如果想记录到文件，只需要在application.properties文件指定logging.path或者logging.file即可。我按照配置，在配置文件，追加了配置
　　

        logging.path=D:/tmp
        logging.file=spring-boot.log

　　然后，启动应用，启动成功后，控制台输出了日志，去查看日志文件，然而，日志文件里没有日志！

# 问题定位和排查

　　奇了怪了，难道是我拼写错误？应该不至于犯这么低等级的错误吧，在网上找了文章，复制文章里的配置，打开bc，对比结果是一毛一样的。  
　　然后，我又想是否application.properties没生效？那我改个端口吧，看下是否端口修改成功。改了端口，启动信息里显示的是新的端口，那配置文件是生效了的  
　　还是不行，没辙了，只能靠万能的谷歌了，谷歌出来的东西，全是教如何配置日志，很多都是一大坨文字、一大坨代码，长的都差不多。  
　　看了半天，没看到什么有价值的东西，还看的晕晕沉沉的，算了明天再处理。  

　　第二天，头脑清醒，尝试了下“新建项目”，配置文件还是用的老项目的，结果还是没有写入到日志；之后配置全部复制网上的一篇文章内的配置，一试，果然是可以了。既然日志能输出了，那转正的原因是什么呢？   
　　仔细对比了两边的配置，才发现，是老的配置，同时配置了logging.path和logging.file，对这两个属性，我本来以为是前者是路径，后者是路径后面的文件名，哪知道，并不是这样的，这两者互斥。  
　　然后，在官方文档上，也看到描述

> By default, Spring Boot logs only to the console and does not write log files. If you want to write log files in addition to the console output, you need to set a logging.file or logging.path property (for example, in your application.properties).

　　注意 logging.file 和 logging.path 中间的 or



# 其他几种情况的尝试

## 以"/"号结尾，会替换成"#"小尾巴

　　假如在application.properties，配置了logging.path为 D:/tmp/spring-boot.log/  
会发生很奇怪的事情，它会在D盘下，创建tmp目录(若不存在)，在tmp目录下，会创建"spring-boot.log#"目录，是的，没看错，是以#替换/的

## Path和File同时存在，file胜

　　假如application.properties，同时配置了logging.path和logging.file，结果是以logging.file为准  
   假如文件路径是精确的，例如 logging.file=D:/tmp/spring-boot.log，那日志就会写到spring-boot.log  
   假如文件路径是不精确的，例如 logging.file=spring-boot.log，那日志就会写到当前项目的路径下 spring-boot.log，跟项目的src同一级别

## application.properties和logback-spring.xml，logback-spring.xml胜

　　假如application.properties，同时配置了logging.path和logging.file；  
并且配置了logback-spring.xml文件，logback-spring.xml里也配置了日志输出到文件。那最终日志输出的文件，是以logback-spring.xml里的为准。  
因为加载顺序  logback.xml > application.properties > logback-spring.xml，最后加载的，会覆盖前者



# 碎碎念

　　好久没有写博客了，看git记录，上一篇文章，还是2018年12月1日写的，距离现在已经将近两个月了。这段时间，发生了两件很重要的事情--离开待了近两年的公司、去了一个陌生的城市。

## 关于离职

　　关于离职，之前就有离职的想法了，我们小组，是公司的业务大组，负责公司最复杂的业务模块，每天晚上下班，走的最晚的必然是我们小组的成员，而我是小组的老人，更是不能幸免；技术上呢，很难得到提高，因为写业务不需要什么技术，所以接触不到什么技术。月初的那几天，又感冒了，非常严重，夜里醒来时，窗外下着雨，看着屋外的雨滴落到窗户上，宁静的夜晚，思绪万千，下一个工作日，到了公司，用虚弱而坚定的声音跟直属领导说我要离职，为了避免挽留，我更是直接说“上海房价太高，我要回家发展”。直属领导听到了很诧异，因为我太突然了。尝试挽留，见我很坚定，只得作罢。

　　然而，离职之前，领导还是希望我把之前排给我的一个大需求，弄完再离开。因为组内的其他成员，排期已经排到2019年了，如果让其他成员接手我的这需求，那他受影响的需求会很多，会导致之前排好日期的需求重新排期，
并且我是最了解那个大需求的开发。就这样，提了离职了，还得写需求。

　　拿离职证明的前一天，送了四本书出去，  
　　　　张嘉佳的《云边有个小卖铺》送给了大黄  
　　　　大冰的《他们最幸福》送给了全哥  
　　　　乐嘉的《写给单身的你》本来是想送春春的，哪知道被他看到书名，果断拒了，后来送给了小组内的新同事  
　　　　《深入JVM虚拟机》送给了我的同桌，格格  
　　送书呢，一是因为要搬家，懒的搬了；二是此次离别，后面很难再见面了，在公司期间，玩的都很愉快，缘分一场，算是留个纪念。

## 关于离开上海

　　关于离开上海，我已经来到上海5年多了，2013年11月27号，还没毕业，一个人，第一次坐和谐号动车，第一次来到了上海，一呆就是整整五年。五年前，我是一张白纸，啥都不会，啥都不懂；五年后，我已经开始独立思考。这期间，发送了太多事情，太多想说的了...