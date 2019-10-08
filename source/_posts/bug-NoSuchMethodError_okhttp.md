---
title: Bug记录 - NoSuchMethodError:okhttp
author: Kairou Zeng
date: 2019/01/28
tags: 
	- bug
	- NoSuchMethodError
	- http
categories:
	- bug
description: 一次okhttp包的NoSuchMethodError
---

## 问题描述

某天，从项目日志看到报了如下错误：

```java
GlobalException:org.springframework.web.util.NestedServletException: Handler dispatch failed; nested exception is java.lang.NoSuchMethodError: okio.BufferedSource.rangeEquals(JLokio/ByteString;)Z
at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1053) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:942) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:998) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:901) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at javax.servlet.http.HttpServlet.service(HttpServlet.java:660) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:875) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at javax.servlet.http.HttpServlet.service(HttpServlet.java:741) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) [tomcat-embed-websocket-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.springframework.boot.actuate.web.trace.servlet.HttpTraceFilter.doFilterInternal(HttpTraceFilter.java:90) [spring-boot-actuator-2.1.0.RELEASE.jar:2.1.0.RELEASE]
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:92) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:93) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.springframework.cloud.sleuth.instrument.web.ExceptionLoggingFilter.doFilter(ExceptionLoggingFilter.java:48) [spring-cloud-sleuth-core-2.0.2.RELEASE.jar:2.0.2.RELEASE]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at brave.servlet.TracingFilter.doFilter(TracingFilter.java:86) [brave-instrumentation-servlet-5.4.3.jar:na]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.filterAndRecordMetrics(WebMvcMetricsFilter.java:154) [spring-boot-actuator-2.1.0.RELEASE.jar:2.1.0.RELEASE]
at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.filterAndRecordMetrics(WebMvcMetricsFilter.java:122) [spring-boot-actuator-2.1.0.RELEASE.jar:2.1.0.RELEASE]
at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:107) [spring-boot-actuator-2.1.0.RELEASE.jar:2.1.0.RELEASE]
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:199) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:490) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:408) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:770) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1415) [tomcat-embed-core-9.0.12.jar:9.0.12]
at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.12.jar:9.0.12]
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_192]
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_192]
at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.12.jar:9.0.12]
at java.lang.Thread.run(Thread.java:748) [na:1.8.0_192]
Caused by: java.lang.NoSuchMethodError: okio.BufferedSource.rangeEquals(JLokio/ByteString;)Z
at okhttp3.internal.Util.bomAwareCharset(Util.java:431) ~[okhttp-3.8.1.jar:na]
at okhttp3.ResponseBody$BomAwareReader.read(ResponseBody.java:249) ~[okhttp-3.8.1.jar:na]
at com.google.gson.stream.JsonReader.fillBuffer(JsonReader.java:1295) ~[gson-2.8.5.jar:na]
at com.google.gson.stream.JsonReader.nextNonWhitespace(JsonReader.java:1333) ~[gson-2.8.5.jar:na]
at com.google.gson.stream.JsonReader.doPeek(JsonReader.java:549) ~[gson-2.8.5.jar:na]
at com.google.gson.stream.JsonReader.peek(JsonReader.java:425) ~[gson-2.8.5.jar:na]
at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:207) ~[gson-2.8.5.jar:na]
at retrofit2.converter.gson.GsonResponseBodyConverter.convert(GsonResponseBodyConverter.java:39) ~[converter-gson-2.4.0.jar:na]
at retrofit2.converter.gson.GsonResponseBodyConverter.convert(GsonResponseBodyConverter.java:27) ~[converter-gson-2.4.0.jar:na]
at retrofit2.ServiceMethod.toResponse(ServiceMethod.java:122) ~[retrofit-2.4.0.jar:na]
at retrofit2.OkHttpCall.parseResponse(OkHttpCall.java:217) ~[retrofit-2.4.0.jar:na]
at retrofit2.OkHttpCall.execute(OkHttpCall.java:180) ~[retrofit-2.4.0.jar:na]
at com.kingdee.kchain.fabric.ca.CaTemplate.callForResult(CaTemplate.java:110) ~[fabric-ca-sdk-1.2.0.M10.jar:na]
at com.kingdee.kchain.fabric.ca.CaTemplate.enroll(CaTemplate.java:76) ~[fabric-ca-sdk-1.2.0.M10.jar:na]
at com.kingdee.kchain.fabric.ca.CaTemplate.<init>(CaTemplate.java:44) ~[fabric-ca-sdk-1.2.0.M10.jar:na]
at com.kingdee.kchain.usersystem.config.CaConfig.lambda$getCaTemplate$0(CaConfig.java:32) ~[classes/:na]
at java.util.concurrent.ConcurrentHashMap.computeIfAbsent(ConcurrentHashMap.java:1660) ~[na:1.8.0_192]
at com.kingdee.kchain.usersystem.config.CaConfig.getCaTemplate(CaConfig.java:29) ~[classes/:na]
at com.kingdee.kchain.usersystem.service.CaService.caUserRegisterAndEnroll(CaService.java:58) ~[classes/:na]
at com.kingdee.kchain.usersystem.service.CaService.newUserCa(CaService.java:91) ~[classes/:na]
at com.kingdee.kchain.usersystem.service.KdUserService.newDefauleUserCa(KdUserService.java:316) ~[classes/:na]
at com.kingdee.kchain.usersystem.service.KdUserService.newUserFromKdUser(KdUserService.java:303) ~[classes/:na]
at com.kingdee.kchain.usersystem.service.KdUserService.kdUserActivate(KdUserService.java:132) ~[classes/:na]
at com.kingdee.kchain.usersystem.service.KdUserService.kdCloudLogin(KdUserService.java:81) ~[classes/:na]
at com.kingdee.kchain.usersystem.service.KdUserService$$FastClassBySpringCGLIB$$36f547ee.invoke(<generated>) ~[classes/:na]
at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:746) ~[spring-aop-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163) ~[spring-aop-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.validation.beanvalidation.MethodValidationInterceptor.invoke(MethodValidationInterceptor.java:119) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:688) ~[spring-aop-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at com.kingdee.kchain.usersystem.service.KdUserService$$EnhancerBySpringCGLIB$$76723ce.kdCloudLogin(<generated>) ~[classes/:na]
at com.kingdee.kchain.usersystem.web.KdUserController.kdCloudLogin(KdUserController.java:35) ~[classes/:na]
at sun.reflect.GeneratedMethodAccessor194.invoke(Unknown Source) ~[na:na]
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_192]
at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_192]
at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:215) ~[spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:142) ~[spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:102) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:800) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1038) ~[spring-webmvc-5.1.2.RELEASE.jar:5.1.2.RELEASE]
... 59 common frames omitted
```

okio报的NoSuchMethodError的错误，想起来前两个星期也刚刚遇到了类似的问题，当时是因为fabric-ca-sdk中引入的okhttp的version为3.8.1，后将okhttp升级到3.12.0后解决该问题。

专门去另个一个项目里将错误重现出来：

```java
org.springframework.web.util.NestedServletException: Handler dispatch failed; nested exception is java.lang.NoSuchMethodError: okhttp3.HttpUrl.get(Ljava/lang/String;)Lokhttp3/HttpUrl;
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1053) ~[spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:942) ~[spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1005) [spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:908) [spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:660) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:882) [spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) [tomcat-embed-websocket-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.springframework.boot.actuate.web.trace.servlet.HttpTraceFilter.doFilterInternal(HttpTraceFilter.java:90) [spring-boot-actuator-2.1.2.RELEASE.jar:2.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:92) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:93) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.springframework.cloud.sleuth.instrument.web.ExceptionLoggingFilter.doFilter(ExceptionLoggingFilter.java:48) [spring-cloud-sleuth-core-2.0.2.RELEASE.jar:2.0.2.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at brave.servlet.TracingFilter.doFilter(TracingFilter.java:86) [brave-instrumentation-servlet-5.4.3.jar:na]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.filterAndRecordMetrics(WebMvcMetricsFilter.java:117) [spring-boot-actuator-2.1.2.RELEASE.jar:2.1.2.RELEASE]
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:106) [spring-boot-actuator-2.1.2.RELEASE.jar:2.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:199) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:490) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:408) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:834) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1417) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) [na:1.8.0_73]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) [na:1.8.0_73]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.14.jar:9.0.14]
	at java.lang.Thread.run(Thread.java:745) [na:1.8.0_73]
Caused by: java.lang.NoSuchMethodError: okhttp3.HttpUrl.get(Ljava/lang/String;)Lokhttp3/HttpUrl;
	at retrofit2.Retrofit$Builder.baseUrl(Retrofit.java:458) ~[retrofit-2.5.0.jar:na]
	at com.kingdee.kchain.fabric.ca.CaTemplate.<init>(CaTemplate.java:39) ~[fabric-ca-sdk-1.2.0.M11.jar:na]
	at com.kingdee.kchain.digitalidentity.config.CaConfig.lambda$getCaTemplate$1(CaConfig.java:29) ~[classes/:na]
	at java.util.concurrent.ConcurrentHashMap.computeIfAbsent(ConcurrentHashMap.java:1660) ~[na:1.8.0_73]
	at com.kingdee.kchain.digitalidentity.config.CaConfig.getCaTemplate(CaConfig.java:26) ~[classes/:na]
	at com.kingdee.kchain.digitalidentity.dao.CaClient.registerAndEnroll(CaClient.java:34) ~[classes/:na]
	at com.kingdee.kchain.digitalidentity.service.DigitalIdentityService.registerByProxy(DigitalIdentityService.java:97) ~[classes/:na]
	at com.kingdee.kchain.digitalidentity.service.DigitalIdentityService$$FastClassBySpringCGLIB$$e2dfa5a.invoke(<generated>) ~[classes/:na]
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:749) ~[spring-aop-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163) ~[spring-aop-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.validation.beanvalidation.MethodValidationInterceptor.invoke(MethodValidationInterceptor.java:119) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:688) ~[spring-aop-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at com.kingdee.kchain.digitalidentity.service.DigitalIdentityService$$EnhancerBySpringCGLIB$$f1f7222c.registerByProxy(<generated>) ~[classes/:na]
	at com.kingdee.kchain.digitalidentity.controller.DigitalIdentityController.registerByProxy(DigitalIdentityController.java:54) ~[classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_73]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_73]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_73]
	at java.lang.reflect.Method.invoke(Method.java:497) ~[na:1.8.0_73]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:189) ~[spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138) ~[spring-web-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:102) ~[spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895) ~[spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:800) ~[spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1038) ~[spring-webmvc-5.1.4.RELEASE.jar:5.1.4.RELEASE]
	... 58 common frames omitted
```

## 问题解决

依旧是版本问题，将低版本依赖去掉，并添加一个okhttp依赖

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.12.0</version>
</dependency>
```

## 相关链接

[OkHttp官网](http://square.github.io/okhttp/)
