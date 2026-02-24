---
title: 你假笨JVM参数 – 006 ExplicitGCInvokesConcurrent
author: Bridge Li
type: post
date: 2017-12-23T11:14:14+00:00

categories:
  - 你假笨说JVM参数
tags:
  - ExplicitGCInvokesConcurrent
  - JVM

---
你假笨的第六次分享：

序号：006  
时间：2017-07-31  
参数：-XX:ExplicitGCInvokesConcurrent  
含义：  
Enables invoking of concurrent GC by using the System.gc() request.  
This option is disabled by default and can be enabled only together with the -XX:+UseConcMarkSweepGC option.  
System.gc()是正常FULL GC，会STW  
打开此参数后，在做System.gc()时会做background模式CMS GC，即并行FULL GC，可提高FULL GC效率  
注，该参数在允许systemGC且使用CMS GC时有效  
举例：  
-XX:+ExplicitGCInvokesConcurrent  
相关参数：  
-XX:DisableExplicitGC 控制是否允许System.gc()，默认允许

小程序截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/ExplicitGCInvokesConcurrent_JVMPocket-254x1024.png" alt="" width="254" height="1024" class="alignnone size-medium wp-image-497" /> 

分享记录：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/ExplicitGCInvokesConcurrent-233x1024.png" alt="" width="233" height="1024" class="alignnone size-medium wp-image-496" />