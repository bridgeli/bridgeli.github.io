---
title: 你假笨说JVM参数 – 002 StringTableSize
author: Bridge Li
type: post
date: 2017-11-05T06:53:11+00:00

categories:
  - 你假笨说JVM参数
tags:
  - CPU负载
  - intern
  - JVM
  - StringTableSize

---
没想到距离第一次整理你假笨的分享已经过去两个多月了，近期会继续整理一系列你假笨关于JVM参数的分享，下面是第二次：

序号：002  
时间：2017-07-14  
参数：-XX:StringTableSize  
含义：Number of buckets in the interned String table  
String.intern() 被调用时会往 Hashtable 插入一个 String（若该 String 不存在），这里的 Table 就是 StringTable，此参数就是这个 StringTable 的大小，若此参数设置过小，明显的问题就是过多的hash碰撞，造成在查找字符串时比较消耗 CPU 资源  
JDK 1.6 起，当冲突次数超过 100 次会自动 rehash，即便如此，若此参数设置过小会导致不断的 rehash，依然会过度消耗 CPU 资源  
建议将此参数设置的值稍大一些，以减少 hash 冲突

使用方法：-XX:ReservedCodeCacheSize=__

小程序截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/11/JVMPocket_StringTableSize-576x1024.jpg" alt="" width="576" height="1024" class="alignnone size-medium wp-image-424" /> 

分享记录：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/11/StringTableSize-260x1024.png" alt="" width="260" height="1024" class="alignnone size-medium wp-image-425" /> 

感谢你假笨