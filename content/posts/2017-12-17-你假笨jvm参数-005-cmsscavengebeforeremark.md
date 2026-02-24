---
title: 你假笨JVM参数 – 005 CMSScavengeBeforeRemark
author: Bridge Li
type: post
date: 2017-12-17T09:22:35+00:00

categories:
  - 你假笨说JVM参数
tags:
  - CMSScavengeBeforeRemark
  - JVM

---
你假笨的第五次分享：

序号：005  
时间：2017-07-24  
参数：-XX:CMSScavengeBeforeRemark  
含义：  
Enable scavenging attempts before the CMS remark step.  
开启或关闭在CMS重新标记阶段之前的清除（YGC）尝试  
CMS并发标记阶段与用户线程并发进行，此阶段会产生已经被标记了的对象又发生变化的情况，若打开此开关，可在一定程度上降低CMS重新标记阶段对上述“又发生变化”对象的扫描时间，当然，“清除尝试”也会消耗一些时间  
注，开启此开关并不会保证在标记阶段前一定会进行清除操作

小程序截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/CMSScavengeBeforeRemark_JVMPocket-334x1024.png" alt="" width="334" height="1024" class="alignnone size-medium wp-image-474" /> 

分享记录：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/CMSScavengeBeforeRemark-228x1024.png" alt="" width="228" height="1024" class="alignnone size-medium wp-image-473" />