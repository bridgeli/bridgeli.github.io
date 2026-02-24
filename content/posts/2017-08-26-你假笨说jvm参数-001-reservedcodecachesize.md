---
title: 你假笨说JVM参数 – 001 ReservedCodeCacheSize
author: Bridge Li
type: post
date: 2017-08-26T08:24:43+00:00

categories:
  - 你假笨说JVM参数
tags:
  - JVM
  - ReservedCodeCacheSize

---
因为之前看过周志明《深入理解Java虚拟机JVM高级特性和最佳实践》，而对JVM的一些东西感兴趣，感觉挺好玩的，前段时间有幸加了阿里寒泉子的微信（现在应该是前阿里了），而加入了一个你假笨建的一个JVM参数交流群，你假笨在里面做过几次分享，看到有小伙伴整理笔记，表示赞同。因为俗话说好记性不如烂笔头，何况自己记性并不怎么好，以下是第一次的分享。另外虽然这些东西平时可能用不到，但当实际出问题的时候不懂这些肯定是束手无策，所以多看看总没有坏处

序号：001  
时间：2017-07-13  
参数：-XX:ReservedCodeCacheSize  
含义：Reserved code cache size (in bytes) &#8211; maximum code cache size  
用于设置Code Cache大小，JIT编译的代码都放在Code Cache中，若Code Cache空间不足则JIT无法继续编译，并且会去优化，比如编译执行改为解释执行，由此，性能会降低  
等价参数：-Xmaxjitcodesize  
使用方法：-XX:ReservedCodeCacheSize=__  
小程序截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/08/JVMPocket-ReservedCodeCacheSize-488x1024.png" alt="" width="488" height="1024" class="alignnone size-medium wp-image-395" /> 

分享记录：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/08/ReservedCodeCacheSize-222x1024.png" alt="" width="222" height="1024" class="alignnone size-medium wp-image-396" /> 

文中小程序截图，是你假笨为了方便大家查询JVM参数而开发的一个小程序，大家可以搜索：JVMPocket添加到自己小程序中，没事的翻翻这些参数也挺好，另外JVMPocket里面有一个签到功能，你假笨会每天设置一个签到的目标人数，达到目标第二天就会在群里面分享一个JVM参数，所以大家可以每天签到，并加到你假笨建的的JVM参数交流群里面，当面听大神的教导。