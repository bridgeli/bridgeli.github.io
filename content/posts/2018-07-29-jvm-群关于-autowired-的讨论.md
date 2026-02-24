---
title: JVM 群关于 Autowired 的讨论
author: Bridge Li
type: post
date: 2018-07-29T06:44:53+00:00

categories:
  - Java
tags:
  - Autowired
  - Resource
  - spring

---
前一段时间 JVM 群有人遇到了一个 stackoverflow 的问题，引发了一个关于 Autowired 的讨论，由于我做的项目可能比较小，并没有遇到过，但感觉这也许就是一个坑，记录下来  
，如果谁有遇到这个问题，说不定就有帮助。

下面我会贴出来群里面的讨论，如果不想看，直接看我的得出的结论，所以 TL;DR 版：

spring 中依赖注入有两个注解：Autowired 和 Resource。Resource 的注入的时候是 byName，而 Autowired 注入的时候是 byType，所以平时并没有很大的区别，但 Autowired 和 getBean(Object.class) 会导致栈很深，有可能会导致 stackoverflow，所以如果遇到这个栈很深，不使用 Autowired 会缓解一下，也就是根据我的理解，在项目中能用 Resource 还是用 Resource 比较好。

原版群里的详细讨论如下图（至于深层次的原因，大家可以根据源码自己跟进看一下）：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource1-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-551" /> 

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource2-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-552" /> 

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource3-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-553" /> 

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource4-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-554" /> 

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource5-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-555" /> 

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource6-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-556" /> 

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource7-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-557" /> 

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource8-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-558" /> 

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/Autowired_Resource9-768x1451.png" alt="" width="768" height="1451" class="alignnone size-medium wp-image-559" /> 

里面 深圳-随缘-颉 贴出的一张图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/07/spring_papulateBean-768x445.jpeg" alt="" width="768" height="445" class="alignnone size-medium wp-image-561" /> 

感谢这一群小伙伴，如果有信息泄漏，很抱歉。

另外曾经有朋友因为 JVM 的存在，就认为 Java 没有内存泄漏的问题，这是对 Java 有多大的误解啊。