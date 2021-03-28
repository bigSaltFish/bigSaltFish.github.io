---
layout: default
title: Gson解析 Expected BEGIN_OBJECT but was NUMBER at line 1 column 5 path $
#excerpt: 
---



　　很久没使用Gson了，之前都是使用阿狸的fastJson。使用gson的时候，莫名其妙的报了这个错  

```java
com.google.gson.JsonSyntaxException: java.lang.IllegalStateException: Expected BEGIN_OBJECT but was NUMBER at line 1 column 5 path $
    at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:226)
    at com.google.gson.Gson.fromJson(Gson.java:927)
    at com.google.gson.Gson.fromJson(Gson.java:892)
    at com.google.gson.Gson.fromJson(Gson.java:841)
    at com.google.gson.Gson.fromJson(Gson.java:813)
    at dfh.model.OnlinePayment.setCustomParamsVo(OnlinePayment.java:266)
    at dfh.skydragon.webservice.ThirdDeopsitServiceWS.payRiskCtrl(ThirdDeopsitServiceWS.java:467)
    at dfh.skydragon.webservice.ThirdDeopsitServiceWS.doThirdDeopsit(ThirdDeopsitServiceWS.java:183)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
    at java.lang.reflect.Method.invoke(Unknown Source)
    at org.apache.axis2.rpc.receivers.RPCUtil.invokeServiceClass(RPCUtil.java:212)
    at org.apache.axis2.rpc.receivers.RPCMessageReceiver.invokeBusinessLogic(RPCMessageReceiver.java:117)
    at org.apache.axis2.receivers.AbstractInOutMessageReceiver.invokeBusinessLogic(AbstractInOutMessageReceiver.java:40)
    at org.apache.axis2.receivers.AbstractMessageReceiver.receive(AbstractMessageReceiver.java:114)
    at org.apache.axis2.engine.AxisEngine.receive(AxisEngine.java:181)
    at org.apache.axis2.transport.http.HTTPTransportUtils.processHTTPPostRequest(HTTPTransportUtils.java:172)
    at org.apache.axis2.transport.http.AxisServlet.doPost(AxisServlet.java:146)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:647)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:728)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:305)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:210)
    at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:51)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:243)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:210)
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:222)
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:123)
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:171)
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:100)
    at org.apache.catalina.valves.AccessLogValve.invoke(AccessLogValve.java:953)
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:118)
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:408)
    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1041)
    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:603)
    at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:310)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
    at java.lang.Thread.run(Unknown Source)
Caused by: java.lang.IllegalStateException: Expected BEGIN_OBJECT but was NUMBER at line 1 column 5 path $
    at com.google.gson.stream.JsonReader.beginObject(JsonReader.java:385)
    at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:215)
    ... 39 more
```

　　出现解析出错，一般就是格式的问题了。仔细看出错内容的描述“Expected BEGIN_OBJECT but was NUMBER at line 1 column 5 ”     

　　大体的意思，是期待是对象开头（即"{"开头），但是，实际是Number(是数字)。现在很明确了，就是应该传入花括号开头的字符串进去，但是，实际传入的却是数字。打个断点，debug进去，一看，果然是数据库的字段填错了