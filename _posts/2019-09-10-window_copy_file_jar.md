---
layout: default
title: window 拷贝文件和打包jar
#excerpt: 

---

一个鬼项目，是maven项目，但是，同步代码需要使用同步脚本。标准的maven配置下，生成的class会在target目录下，但是，用脚本更新，必须要把class放到WEB-INF/class文件夹下面去，设置了编译器的编译class指向，clas倒是运过来了，项目里跑个main函数都报莫名其妙的错，算了吧，还是自己写个复制脚本

同步脚本.bat

```bash
:: 删除文件
rmdir /s/q D:\dev_soft\eclipse_workspace\data_center\WebContent\WEB-INF\classes\

xcopy D:\dev_soft\eclipse_workspace\data_center\target\classes D:\dev_soft\eclipse_workspace\data_center\WebContent\WEB-INF\classes\ /e

:: 移动jar包到lib文件夹下面
mvn dependency:copy-dependencies -DoutputDirectory=WebContent/WEB-INF/lib   -DincludeScope=runtime
```



  

