---
layout: default
title: 记一次XSS注入问题
#excerpt: 

---

　　业务人员突然反馈后台的数据显示不正常，让技术找原因。  

# 1.排查问题

　　我在排查问题的时候，发现是用户在提交的时候，后端没有对内容进行XSS过滤，导致数据库中存入了脚本，包含脚本的内容返回给了前台，绘画页面的时候，因为脚本的标签没闭合，导致了绘画中断，所以页面显示不正常。  
　　数据库中存储的是这样的  

  ![包含脚本的内容]({{site.url}}/assets/2019-10-22-js_injeck/xss_wrong_content.jpg)

# 2.问题分析

　　正好对XSS挺感兴趣的，就来看看，存入了js脚本后，到底是如何进行恶意攻击的。  

　　准备一个本地的web项目，用来测试，在登录页面加上图中的内容，观察浏览器的NetWork

![network]({{site.url}}/assets/2019-10-22-js_injeck/xss_wrong_network.jpg)

　　上图中，有三个异常的请求

## 异常请求1

　　第一个OrAI请求，是301转发，因为我本地是http的，它的网站是https的，请求它的资源，都得是https的，所以会301到https

![第一个OrAI请求]({{site.url}}/assets/2019-10-22-js_injeck/xss_wrong_network_1.jpg)

## 异常请求2

　　第二个OrAI请求，返回的是一段js脚本

![第二个OrAI请求]({{site.url}}/assets/2019-10-22-js_injeck/xss_wrong_network_2.jpg)

![第二个OrAI请求]({{site.url}}/assets/2019-10-22-js_injeck/xss_wrong_network_2_response.jpg)

　　这段js很简单，就是获取url和cookie，然后伪装请求图片，在“图片”链接后面带上参数，发送给对方。有了url和cookie，那对方就可以进入后台为所欲为了

## 异常请求3

　　第三个请求，就是发送我方数据给对方了，很直观

![第二个请求]({{site.url}}/assets/2019-10-22-js_injeck/xss_wrong_network_3.jpg)

　　从网络请求可以看出，把数据送到了一个xss平台  

# 3.问题处理和防范

　　对所有后端接收到的参数，都要做XSS和SQL过滤。已入库的数据，要进行更正

　　但是，我看数据加入的数据，已经很长时间了，按道理他应该早就提现了的，咋一直没听到相关资安的问题呢？后来才反应过来，我们后台是有做ip限制的，只有在白名单的ip才可以打开，这样就一定程度的避免了恶意提现的情况