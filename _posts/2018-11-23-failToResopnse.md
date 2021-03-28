---
layout: default
title: httpClient fail to respond
#excerpt: 

---

```java
  org.apache.http.NoHttpResponseException: www.abc.com:port failed to respond
```
　　这周在做一个功能，在调用外部接口的时候，莫名其妙的报这个错误。使用的类是很常见的HttpClient。网上看了其他文章，都是说什么设置“keepAlive”属性，试过了，没用；后面又怀疑服务方（www.abc.com），又证实服务方是ok的。之后又猜测是不是HttpClient的问题，就使用okHttp来调接口，使用okHttp确实是没 fail to resond的异常了，不过，有了新的异常 connect timed out

```java
java.net.SocketTimeoutException: connect timed out
    at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
    at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:85)
    at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:345)
    at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
    at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
    at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
    at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
    at java.net.Socket.connect(Socket.java:589)
    at okhttp3.internal.Platform.connectSocket(Platform.java:126)
    at okhttp3.internal.io.RealConnection.connectSocket(RealConnection.java:140)
    at okhttp3.internal.io.RealConnection.connect(RealConnection.java:111)
    at okhttp3.internal.http.StreamAllocation.findConnection(StreamAllocation.java:188)
    at okhttp3.internal.http.StreamAllocation.findHealthyConnection(StreamAllocation.java:127)
    at okhttp3.internal.http.StreamAllocation.newStream(StreamAllocation.java:97)
```

　　最后还是经验老到的一个同事指出了错误：端口错了！网站www.abc.com提供了多个端口，写和读不是同一个端口，这个功能已经上线了，然而，我在上面开发新功能的时候才发现是有问题的，也就是之前那哥们根本就盲写代码！并且上线了！尼玛，也是无语了！