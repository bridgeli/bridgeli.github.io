---
title: 你假笨JVM参数 – 004 MaxTenuringThreshold
author: Bridge Li
type: post
date: 2017-12-10T07:32:14+00:00

categories:
  - 你假笨说JVM参数
tags:
  - JVM
  - MaxTenuringThreshold
  - 对象年龄

---
你假笨的第四次分享：

序号：004  
时间：2017-07-21  
参数：-XX:MaxTenuringThreshold  
含义：  
Sets the maximum tenuring threshold for use in adaptive GC sizing.  
The largest value is 15.  
The default value is 15 for the parallel (throughput) collector, and 6 for the CMS collector.  
在可自动调整对象晋升老年代年龄阈值的GC中，该参数用于设置上述年龄阈值的最大值  
参数值最大为15  
Parallel Scavenge中默认值为15，CMS中默认值为6，G1中默认值为15

小程序截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/MaxTenuringThreshold_JVMPocket-290x1024.png" alt="" width="290" height="1024" class="alignnone size-medium wp-image-469" /> 

分享记录：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/MaxTenuringThreshold-298x1024.png" alt="" width="298" height="1024" class="alignnone size-medium wp-image-468" />