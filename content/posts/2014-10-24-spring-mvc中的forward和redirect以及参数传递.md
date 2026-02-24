---
title: Spring mvc中的forward和redirect以及参数传递
author: Bridge Li
type: post
date: 2014-10-24T09:49:46+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
---
1. forward和redirect  
大家都知道servlet在处理完业务逻辑返回时有两种方法forward和redirect，他们的差异相信不用我再多做解释（如果不知道的请自行谷歌，哪怕是百度也可以），而Spring mvc是对servlet的一种封装，那spring mvc默认采用的是哪一种呢？我们是否可以自己选择采用哪一种方式返回呢？还有我之前在用spring mvc 都是返回到某一个view，它是否可以访问另一个controller呢？针对第一个问题，我们可以看一下spring mvc 的配置文件便知分晓：  
```  
<property name="viewResolvers">
    <list>
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/pages/"/>
            <property name="suffix" value=".jsp"/>
        </bean>
    </list>
</property>  
```  
因为WEB-INF是一个受保护的目录，客户端是访问不到的，只能通过服务器端访问，根据forward和redirect的区别，我们很容易看到是forward的方式返回的，针对第二个和第三个问题，其实也很简单，我们只需要让该controller返回String即可，然后在方法的最后 return &#8220;redirect:/game&#8221;;或者return &#8220;forward:/game&#8221;;即 redirect或者forward + “：” + “/” + controller的路径或者view的名字即可。

2. 参数传递  
因为公司的项目是用Spring mvc开发的，发现参数传递除了通过model，还可以通过 RedirectAttributes，据说参数的传递和跳转的URL后面带的值会有一定的关系，这个我没具体测试，感兴趣的可以自己测测。

总结：这篇文章是近期学到的知识点，以前感觉自己会用Spring mvc了，今天才发现还有好多Spring mvc的特性不知道，值得自己去探索啊！