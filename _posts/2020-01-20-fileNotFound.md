---
layout: default
title: 记一次FileNotFoundException异常
#excerpt: 

---

　　同事提交了代码，就回去过春节了，今天，业务人员来说他昨天修改的东西有问题，要我处理。看了下日志，发现是这个很easy的异常

	java.io.FileNotFoundException: \excel-templates\123 (系统找不到指定的路径。)

　　看错误信息，就知道是xls的模版在服务器上没有被程序获取到，又看了下git上是有的，那绝壁是运维哥哥的锅了。

　　果断吊运维去(这个docker环境下的项目，开发看不到服务器上的代码，没权限)

　　运维排查了下，发现，服务器上是有xls模版的，又把问题丢给我了。



# 提交模版文件的项目错误

　　奇了怪了，服务器上有模版的，git也有提交记录的，为什么会这样呢？

　　带着疑惑的我，本地试了试，发现，也是会报错的，fileNotFoundException，心里顿时MMP，这太扯了吧，那哥们都不测的，就提交上去了？

　　仔细看了下他的代码，发现，他本地是有测试的(因为本地测试代码还提交上去了的...)，不过，他测的是磁盘绝对路径的方式，例如 "c://temp/abc.xls"，这种方式是没有问题的。

　　但是，代码里实际使用，是使用classpath相对路径去取的，例如："classpath:excel-templates/abc.xls"，我们项目是父子项目，他只在父项目提交了xls模版，没有在common项目里提交模版。

　　问题定位了，在common里上传模版，本地测试，一切正常。

　　ok，提交代码，线上重启。



# ResourceUtils.getFile() 未获取预期值

　　本以为没事了，结果，自测的时候，发现，仍然是报错的，错误信息仍然是 fileNotFoundException

　　去服务器看了下，common下面的模版是存在的

　　OMG，惊呆了，有点怀疑人生



　　只能发动绝招了-面向谷歌编程

　　但是，也没有收获到非常有价值的信息

　　项目的代码，很简单的一行代码，使用到了spring的util类

```java
public static FileInputStream getTemplates(String tempName) throws FileNotFoundException {
        return new FileInputStream(ResourceUtils.getFile("classpath:excel-templates/"+tempName));
}
```



# 曲线救国--更换获取模版方案

　　无奈之下，只能曲线救国--换新的获取方式。使用getResourceAsStream的方式，结果，真的正常了...

　　下面附上代码：

```java
public static InputStream getTemplates(String tempName) {
    InputStream resourceAsStream = TemplateFileUtil.class.getResourceAsStream("/excel-templates/" + tempName);
    return resourceAsStream;
}
```



# 两者获取模版文件的差异是什么？

　　这个得去深扒源码了，我太难了---