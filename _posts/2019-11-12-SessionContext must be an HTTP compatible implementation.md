---
layout: default
title: SessionContext must be an HTTP compatible implementation
#excerpt: 

---

　　使用SpringBootTest和shiro结合时，发生的异常。具体信息：

```java
org.springframework.web.util.NestedServletException: Request processing failed; nested exception is java.lang.IllegalArgumentException: SessionContext must be an HTTP compatible implementation.

    at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:982)
    at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:872)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:661)
    at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846)
    at org.springframework.test.web.servlet.TestDispatcherServlet.service(TestDispatcherServlet.java:65)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:742)
    at org.springframework.mock.web.MockFilterChain$ServletFilterProxy.doFilter(MockFilterChain.java:160)
    at org.springframework.mock.web.MockFilterChain.doFilter(MockFilterChain.java:127)
    at org.springframework.test.web.servlet.MockMvc.perform(MockMvc.java:155)
    at com.frank.my_test.MyShiroTest.testController(MyShiroTest.java:78)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
    at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
    at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
    at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
    at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:26)
    at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:75)
    at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:86)
    at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:84)
    at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
    at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:252)
    at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:94)
    at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
    at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
    at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
    at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
    at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
    at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61)
    at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70)
    at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
    at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:191)
    at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
    at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
    at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
    at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
Caused by: java.lang.IllegalArgumentException: SessionContext must be an HTTP compatible implementation.
    at org.apache.shiro.web.session.mgt.ServletContainerSessionManager.createSession(ServletContainerSessionManager.java:103)
    at org.apache.shiro.web.session.mgt.ServletContainerSessionManager.start(ServletContainerSessionManager.java:64)
    at org.apache.shiro.mgt.SessionsSecurityManager.start(SessionsSecurityManager.java:152)
    at org.apache.shiro.subject.support.DelegatingSubject.getSession(DelegatingSubject.java:340)
    at org.apache.shiro.subject.support.DelegatingSubject.getSession(DelegatingSubject.java:316)
    at org.apache.shiro.mgt.DefaultSubjectDAO.mergePrincipals(DefaultSubjectDAO.java:207)
    at org.apache.shiro.mgt.DefaultSubjectDAO.saveToSession(DefaultSubjectDAO.java:165)
    at org.apache.shiro.mgt.DefaultSubjectDAO.save(DefaultSubjectDAO.java:146)
    at org.apache.shiro.mgt.DefaultSecurityManager.save(DefaultSecurityManager.java:387)
    at org.apache.shiro.mgt.DefaultSecurityManager.createSubject(DefaultSecurityManager.java:354)
    at org.apache.shiro.mgt.DefaultSecurityManager.createSubject(DefaultSecurityManager.java:187)
    at org.apache.shiro.mgt.DefaultSecurityManager.login(DefaultSecurityManager.java:287)
    at org.apache.shiro.subject.support.DelegatingSubject.login(DelegatingSubject.java:260)
    at com.frank.controller.LoginController.login(LoginController.java:35)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205)
    at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:133)
    at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:97)
    at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:827)
    at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:738)
    at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:85)
    at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:967)
    at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:901)
    at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970)
```

　　用谷歌搜了这个错误，貌似发生这个错误的不多。其中有一篇文章，讲的是类型不匹配
　　
```java
org.apache.shiro.mgt.DefaultSecurityManager
```

　　上面的是错误的，应该要引用
　　
```java
org.apache.shiro.web.mgt.DefaultWebSecurityManager
```

　　试了，没效果，又尝试了几次，才知道，原来是需要手动塞subject