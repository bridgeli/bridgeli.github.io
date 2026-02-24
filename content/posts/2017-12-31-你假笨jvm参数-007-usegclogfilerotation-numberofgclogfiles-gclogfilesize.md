---
title: 你假笨JVM参数 – 007 UseGCLogFileRotation NumberOfGCLogFiles GCLogFileSize
author: Bridge Li
type: post
date: 2017-12-31T10:19:25+00:00

categories:
  - 你假笨说JVM参数
tags:
  - GCLogFileSize
  - JVM
  - NumberOfGCLogFiles
  - UseGCLogFileRotation

---
你假笨的第七次分享，也是你假笨在 2017 年的最后一次关于 JVM 的分享：

序号：007  
时间：2017-08-10  
参数：  
-XX:UseGCLogFileRotation  
-XX:NumberOfGCLogFiles  
-XX:GCLogFileSize  
含义：  
这次分享了3个设置滚动记录GC日志的参数  
通过参数-Xloggc:xxx可指定GC日志文件路径  
普通情况下，GC日志文件内容会不断积累，进程重启后日志文件会被覆盖  
这次分享的3个参数在设置-Xloggc参数的前提下有效

-XX:UseGCLogFileRotation  
Enabled GC log rotation, requires -Xloggc.  
打开或关闭GC日志滚动记录功能，要求必须设置 -Xloggc参数

-XX:NumberOfGCLogFiles  
Set the number of files to use when rotating logs, must be >= 1.  
The rotated log files will use the following naming scheme, <filename>.0, <filename>.1, &#8230;, <filename>.n-1.  
设置滚动日志文件的个数，必须大于1  
日志文件命名策略是，<filename>.0, <filename>.1, &#8230;, <filename>.n-1，其中n是该参数的值

-XX:GCLogFileSize  
The size of the log file at which point the log will be rotated, must be >= 8K.  
设置滚动日志文件的大小，必须大于8k  
当前写日志文件大小超过该参数值时，日志将写入下一个文件

注，GC日志最好不滚动输出，因为之前的关键日志可能会被冲掉，日志写入同一个文件更方便分析

相关参数：  
-Xloggc:xxx 指定GC日志文件路径，设置改参数后分享的3个参数有效

相关小程序截图

UseGCLogFileRotation ：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/UseGCLogFileRotation-542x1024.jpg" alt="" width="542" height="1024" class="alignnone size-medium wp-image-503" /> 

NumberOfGCLogFiles ：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/NumberOfGCLogFiles-542x1024.jpg" alt="" width="542" height="1024" class="alignnone size-medium wp-image-502" /> 

GCLogFileSize ：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/GCLogFileSize-542x1024.jpg" alt="" width="542" height="1024" class="alignnone size-medium wp-image-501" /> 

分享记录：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/12/UseGCLogFileRotation_NumberOfGCLogFiles_GCLogFileSize-768x4916.png" alt="" width="768" height="4096" class="alignnone size-medium wp-image-504" />