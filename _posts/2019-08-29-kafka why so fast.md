---
layout: default
title: kafka是如何实现高性能的写入和读取的？
#excerpt: 
---



# 写

　　 页缓存(page cache) + 顺序写入磁盘   



　　 页缓存：操作系统的内存缓存，kafka写入是写入到内存缓存，然后再从内存缓存刷到磁盘   

　　 顺序写入：一般情况，写磁盘是需要寻找适宜的磁盘空间的。kafka改变策略，直接在尾部追加  



# 读

　　 零拷贝技术   



　　 正常情况下，磁盘的信息-->客户端，是需要：   

  1.磁盘到页缓存；    
  2. 页缓存到软件缓存；    
  3. 软件缓存到socket缓存；  
  4. socket缓存到网卡  


kafka直接略过了步骤2和步骤3
