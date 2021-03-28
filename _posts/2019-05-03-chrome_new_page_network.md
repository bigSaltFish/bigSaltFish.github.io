---
layout: default
title: chrome浏览器如何跟踪新开标签的网络请求？
#excerpt: 
---

　　在测试一个东西的时候，它虽然是a链接，但是，是由前端在js里写跳转的。我又必须要知道它的跳转链接，js还加扰了，看不了。最后只能用截屏的方式来捕捉浏览器的地址栏链接(没知识真可怕)  

　　后来谷歌了才知道，chrome自从 Chrome 50 版本就支持这个功能了。  

　　使用方法很简单  

　　1.打开浏览器控制台(F12)  

　　2.使用三点菜单(F1，控制台右上角X号旁边的那个按钮打开 setting 栏)  

![三点菜单操作]({{site.url}}/assets/2019-05-03-chrome_new_page_network/f1.jpg)

　　3. DevTools区域，勾选第一个 Auto-open DevTools for popups  

![三点菜单操作]({{site.url}}/assets/2019-05-03-chrome_new_page_network/auto-open.jpg)