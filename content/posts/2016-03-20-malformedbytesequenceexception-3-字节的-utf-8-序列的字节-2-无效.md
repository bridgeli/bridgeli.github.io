---
title: 'MalformedByteSequenceException: 3 字节的 UTF-8 序列的字节 2 无效'
author: Bridge Li
type: post
date: 2016-03-20T14:23:00+00:00

duoshuo_thread_id:
  - 6.2641401433141E+18
categories:
  - Java
tags:
  - encoding
  - MalformedByteSequenceException
  - 编码

---
今天这篇文章比较简单，写一个老夫近期工作中遇到的一个问题，这个问题困扰了老夫几个月了，虽然借助强大的Google百度了好久，但一直没有彻底解决，一直感觉挺简单一问题，也挺常见的一问题（网上问这个问题的还挺多），怎么就没有一个靠谱点的解决方案呢，刚好上个周呢时间稍有空闲，于是仔细研究了一下，终于找到了问题的根源，然后同事一言点醒梦中人，豁然开朗，一举解决，所以记录一下，先说一下异常：MalformedByteSequenceException: 3 字节的 UTF-8 序列的字节 2 无效，详细的堆栈信息如下：

```

严重: StandardWrapper.Throwable  
org.springframework.beans.factory.BeanDefinitionStoreException: IOException parsing XML document from file [D:J2EEapache-tomcat-7.0.62webappscrm-v1.0WEB-INFclassesspring-kafka-consumer.xml]; nested exception is com.sun.org.apache.xerces.internal.impl.io.MalformedByteSequenceException: 3 字节的 UTF-8 序列的字节 2 无效。  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(XmlBeanDefinitionReader.java:409)  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(XmlBeanDefinitionReader.java:335)  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(XmlBeanDefinitionReader.java:303)  
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:180)  
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:216)  
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:187)  
at org.springframework.web.context.support.XmlWebApplicationContext.loadBeanDefinitions(XmlWebApplicationContext.java:125)  
at org.springframework.web.context.support.XmlWebApplicationContext.loadBeanDefinitions(XmlWebApplicationContext.java:94)  
at org.springframework.context.support.AbstractRefreshableApplicationContext.refreshBeanFactory(AbstractRefreshableApplicationContext.java:129)  
at org.springframework.context.support.AbstractApplicationContext.obtainFreshBeanFactory(AbstractApplicationContext.java:540)  
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:454)  
at org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:658)  
at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:624)  
at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:672)  
at org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:543)  
at org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:484)  
at org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)  
at javax.servlet.GenericServlet.init(GenericServlet.java:158)  
at org.apache.catalina.core.StandardWrapper.initServlet(StandardWrapper.java:1284)  
at org.apache.catalina.core.StandardWrapper.loadServlet(StandardWrapper.java:1197)  
at org.apache.catalina.core.StandardWrapper.allocate(StandardWrapper.java:864)  
at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:134)  
at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:122)  
at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:505)  
at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:170)  
at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:103)  
at org.apache.catalina.valves.AccessLogValve.invoke(AccessLogValve.java:957)  
at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:116)  
at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:423)  
at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1079)  
at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:620)  
at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:316)  
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)  
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)  
at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)  
at java.lang.Thread.run(Thread.java:745)  
Caused by: com.sun.org.apache.xerces.internal.impl.io.MalformedByteSequenceException: 3 字节的 UTF-8 序列的字节 2 无效。  
at com.sun.org.apache.xerces.internal.impl.io.UTF8Reader.invalidByte(UTF8Reader.java:687)  
at com.sun.org.apache.xerces.internal.impl.io.UTF8Reader.read(UTF8Reader.java:408)  
at com.sun.org.apache.xerces.internal.impl.XMLEntityScanner.load(XMLEntityScanner.java:1735)  
at com.sun.org.apache.xerces.internal.impl.XMLEntityScanner.scanData(XMLEntityScanner.java:1234)  
at com.sun.org.apache.xerces.internal.impl.XMLScanner.scanComment(XMLScanner.java:778)  
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanComment(XMLDocumentFragmentScannerImpl.java:1038)  
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl$FragmentContentDriver.next(XMLDocumentFragmentScannerImpl.java:2988)  
at com.sun.org.apache.xerces.internal.impl.XMLDocumentScannerImpl.next(XMLDocumentScannerImpl.java:606)  
at com.sun.org.apache.xerces.internal.impl.XMLNSDocumentScannerImpl.next(XMLNSDocumentScannerImpl.java:117)  
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanDocument(XMLDocumentFragmentScannerImpl.java:510)  
at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:848)  
at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:777)  
at com.sun.org.apache.xerces.internal.parsers.XMLParser.parse(XMLParser.java:141)  
at com.sun.org.apache.xerces.internal.parsers.DOMParser.parse(DOMParser.java:243)  
at com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderImpl.parse(DocumentBuilderImpl.java:347)  
at org.springframework.beans.factory.xml.DefaultDocumentLoader.loadDocument(DefaultDocumentLoader.java:76)  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadDocument(XmlBeanDefinitionReader.java:428)  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(XmlBeanDefinitionReader.java:390)  
&#8230; 35 more

三月 16, 2016 5:41:59 下午 org.apache.catalina.core.StandardWrapperValve invoke  
严重: Allocate exception for servlet springServlet  
com.sun.org.apache.xerces.internal.impl.io.MalformedByteSequenceException: 3 字节的 UTF-8 序列的字节 2 无效。  
at com.sun.org.apache.xerces.internal.impl.io.UTF8Reader.invalidByte(UTF8Reader.java:687)  
at com.sun.org.apache.xerces.internal.impl.io.UTF8Reader.read(UTF8Reader.java:408)  
at com.sun.org.apache.xerces.internal.impl.XMLEntityScanner.load(XMLEntityScanner.java:1735)  
at com.sun.org.apache.xerces.internal.impl.XMLEntityScanner.scanData(XMLEntityScanner.java:1234)  
at com.sun.org.apache.xerces.internal.impl.XMLScanner.scanComment(XMLScanner.java:778)  
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanComment(XMLDocumentFragmentScannerImpl.java:1038)  
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl$FragmentContentDriver.next(XMLDocumentFragmentScannerImpl.java:2988)  
at com.sun.org.apache.xerces.internal.impl.XMLDocumentScannerImpl.next(XMLDocumentScannerImpl.java:606)  
at com.sun.org.apache.xerces.internal.impl.XMLNSDocumentScannerImpl.next(XMLNSDocumentScannerImpl.java:117)  
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanDocument(XMLDocumentFragmentScannerImpl.java:510)  
at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:848)  
at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:777)  
at com.sun.org.apache.xerces.internal.parsers.XMLParser.parse(XMLParser.java:141)  
at com.sun.org.apache.xerces.internal.parsers.DOMParser.parse(DOMParser.java:243)  
at com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderImpl.parse(DocumentBuilderImpl.java:347)  
at org.springframework.beans.factory.xml.DefaultDocumentLoader.loadDocument(DefaultDocumentLoader.java:76)  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadDocument(XmlBeanDefinitionReader.java:428)  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(XmlBeanDefinitionReader.java:390)  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(XmlBeanDefinitionReader.java:335)  
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(XmlBeanDefinitionReader.java:303)  
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:180)  
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:216)  
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:187)  
at org.springframework.web.context.support.XmlWebApplicationContext.loadBeanDefinitions(XmlWebApplicationContext.java:125)  
at org.springframework.web.context.support.XmlWebApplicationContext.loadBeanDefinitions(XmlWebApplicationContext.java:94)  
at org.springframework.context.support.AbstractRefreshableApplicationContext.refreshBeanFactory(AbstractRefreshableApplicationContext.java:129)  
at org.springframework.context.support.AbstractApplicationContext.obtainFreshBeanFactory(AbstractApplicationContext.java:540)  
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:454)  
at org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:658)  
at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:624)  
at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:672)  
at org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:543)  
at org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:484)  
at org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)  
at javax.servlet.GenericServlet.init(GenericServlet.java:158)  
at org.apache.catalina.core.StandardWrapper.initServlet(StandardWrapper.java:1284)  
at org.apache.catalina.core.StandardWrapper.loadServlet(StandardWrapper.java:1197)  
at org.apache.catalina.core.StandardWrapper.allocate(StandardWrapper.java:864)  
at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:134)  
at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:122)  
at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:505)  
at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:170)  
at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:103)  
at org.apache.catalina.valves.AccessLogValve.invoke(AccessLogValve.java:957)  
at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:116)  
at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:423)  
at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1079)  
at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:620)  
at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:316)  
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)  
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)  
at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)  
at java.lang.Thread.run(Thread.java:745)

```

看完堆栈信息，我相信遇到这个问题的同学，已经知道咋回事了。说起来也很简单，就是tomcat跑一个web项目时，报这个异常，导致项目起不来。在仔细看之后，应该是编码问题导致的，对，就是编码的问题了，看报错的那个XML文件编译之后的情况如下：

[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/03/Exception-300x104.png" alt="Exception" width="300" height="104" class="alignnone size-medium wp-image-260" />][1]

果然出现了乱码，虽然是注释以后的地方出现了乱码，但项目依然跑不起来了。用Google百度了一下，天下文章一大抄，你抄我来我抄他，归根到底主要有两种方法来解决这个问题（不是说没有人有靠谱点的解决方案吗？嗯，是没有，但还是有人遇到过，提出了各种解决方案）：  
1. 修改tomcat的startup.bat的代码，说实话这种解决方式真不靠谱，因为这是配置文件乱码，和tomcat的关系能有多大，但老夫还是抱着希望还是要有的，万一解决了呢的态度，试了试，果不其然没有解决，所以就不贴代码了  
2. 把配置文件全部改为GBK的编码，对，就是把XML声明的第一行的UTF-8改成GBK，还别说，真给解决了，但我们的项目设定的都是UTF-8，突然把配置文件改成GBK，总归有点别扭，但别扭归别扭，解决了还是可以的，但这种办法也有一个弊端，就是同事那有可能乱码了  
3. 不是说有两种吗？为什么会出来了3，这是什么情况，这是老夫不得已提出的一种方法，写什么注释，又不是源码注释，这些注释有谁看，大家都会，把那些出现乱码的地方删了不就完事了，嗯，确实完事了，老夫当时就是这么干的，但有注释总归是好事啊，万一有人不懂呢，万一加入的同事，是吧？不能要求所有人都懂啊

好了，下面给出老夫的彻底解决办法，其实很简单，老夫曾经的文章，对，就是[这篇文章][2]中，已经给出解决的办法了，只是老夫当时没在意，所以印象不深刻，很简单，这明显是build的时候出现了编码问题，设定一下build的时候的编码就成了吗，如果懒得看那篇文章，那好，老夫就在这里贴出来解决办法，其实很简单，就是在pom文件中，指定build的编码：

```

<properties>  
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
</properties>

```

什么你还在用Ant，赶紧换Maven吧，Ant那么难用，老夫懒得研究了，如果您知道怎么解决，请留言指出，还有老夫虽然最终解决了这个问题，但说实话，出现这个问题的原因，真的还不是很清楚，因为老夫真的已经把各个地方都设定为：UTF-8了，但他还是出现了乱码，所以如果有同学指定产生的原因，也请留言指出，这里老夫一并谢谢了，谢谢。  
最后的最后，给出老夫的建议：闲着没事的都设定一下这个build的编码吧，这样能保证你build之后的所有的文件的编码都是UTF-8，虽然不设也许不会出问题，但设定了之后一定能给你减少很多麻烦，而且又不费事。

 [1]: https://www.bridgeli.cn/wp-content/uploads/2016/03/Exception.png
 [2]: https://www.bridgeli.cn/archives/173 "持续集成（C I）工具Jenkins入门"