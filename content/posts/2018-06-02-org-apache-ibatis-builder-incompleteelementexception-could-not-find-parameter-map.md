---
title: 'org.apache.ibatis.builder.IncompleteElementException: Could not find parameter map'
author: Bridge Li
type: post
date: 2018-06-02T09:17:30+00:00

categories:
  - Java
tags:
  - IncompleteElementException
  - mybatis

---
上周和同事一块开发一个功能模块，在开发中拉下来同事代码，在测试的时候，突然跑不通了，报错信息如下：

```

org.apache.ibatis.builder.IncompleteElementException: Could not find parameter map java.util.Map

at org.apache.ibatis.builder.MapperBuilderAssistant.getStatementParameterMap(MapperBuilderAssistant.java:320)  
at org.apache.ibatis.builder.MapperBuilderAssistant.addMappedStatement(MapperBuilderAssistant.java:296)  
at org.apache.ibatis.builder.xml.XMLStatementBuilder.parseStatementNode(XMLStatementBuilder.java:109)  
at org.apache.ibatis.session.Configuration.buildAllStatements(Configuration.java:753)  
at org.apache.ibatis.session.Configuration.hasStatement(Configuration.java:723)  
at org.apache.ibatis.session.Configuration.hasStatement(Configuration.java:718)  
at org.apache.ibatis.binding.MapperMethod$SqlCommand.<init>(MapperMethod.java:201)  
at org.apache.ibatis.binding.MapperMethod.<init>(MapperMethod.java:48)  
at org.apache.ibatis.binding.MapperProxy.cachedMapperMethod(MapperProxy.java:59)  
at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:52)  
at com.sun.proxy.$Proxy40.insertSelective(Unknown Source)  
at com.weidai.urge.service.disposition.BidDispositionOrderServiceTest.testInsertSelective(BidDispositionOrderServiceTest.java:48)  
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)  
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)  
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)  
at java.lang.reflect.Method.invoke(Method.java:606)  
at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)  
at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)  
at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)  
at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)  
at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:75)  
at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:86)  
at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:84)  
at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)  
at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:254)  
at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:89)  
at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)  
at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)  
at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)  
at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)  
at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)  
at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61)  
at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70)  
at org.junit.runners.ParentRunner.run(ParentRunner.java:363)  
at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:193)  
at org.junit.runner.JUnitCore.run(JUnitCore.java:137)  
at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)  
at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:51)  
at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:237)  
at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)  
Caused by: java.lang.IllegalArgumentException: Parameter Maps collection does not contain value for java.util.Map  
at org.apache.ibatis.session.Configuration$StrictMap.get(Configuration.java:853)  
at org.apache.ibatis.session.Configuration.getParameterMap(Configuration.java:625)  
at org.apache.ibatis.builder.MapperBuilderAssistant.getStatementParameterMap(MapperBuilderAssistant.java:318)  
&#8230; 39 more

```

看报错信息，也就是自己跑一个 junit test，测试批量插入，突然他就不行了，再看看自己写的批量插入没问题啊，我也没用 map，感觉好奇怪，就网上搜了一下这个问题，原来确实有问题，也确实在这个配置文件中，提示的错误也对，就是提示的位置不对。mybatis 中只要有任何一个地方报错，都无法通过。最后搜了一下发现是刚刚来下来的代码，同事新增了一个方法上将 parameterType 写成了 parameterMap 了

```

<select id="getByMap" parameterMap="java.util.Map" resultMap="baseResultMap">

```

同事之所以大家犯错，有一个原因是在编写配置文件时：

1. 参数他用的 Map；  
2. idea 还会提示 parameterMap；

但是在 mybatis3 中已经不再用这个属性了，所以就报错了，这也要求我们写代码的时候要小心，同时知识也要随着时代更新。

其实这些都不重要，也不值得写篇文章，重要的是我个人有两点小小的感悟，也很短但感觉很有用，想说说：

1. 对于自己提交的代码，大家一定要心里有个基本的认识，像这样的自己的提交的代码质量没有保证，是对同事的严重不负责。不过这个还好，最起码 build 能过，曾经不止一次遇到过连 build 都过不了的代码就被提交的同事，也是服气；

2. 个人建议方法的参数最好不要用 Map，我们都知道 Map 是 KV 结构的数据结构，所以他就有一个问题，别人在调用你的方法的时候不知道可以往 Map 里面放哪些 key，必须读代码才知道，也就是你的代码被没有做到见名知意，还有我猜测如果这位同事不是用 Map 参数也肯定不会写错，犯这个小错误，另外我还遇到过写的方法参数全是 Map 的同事，我表示十万个大写的 服。