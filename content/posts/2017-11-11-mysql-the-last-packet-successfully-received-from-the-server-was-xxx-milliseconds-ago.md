---
title: 'MySQL : The last packet successfully received from the server was XXX milliseconds ago'
author: Bridge Li
type: post
date: 2017-11-11T08:08:13+00:00

categories:
  - MySQL
tags:
  - CommunicationsException
  - connection
  - 数据库
  - 连接

---
14年毕业写完论文没事干的时候，自己玩微信公众号开发，当时想做一个自然语言交互，其实就是想试一下lucene，但是当时建索引的时候偶尔会报这个错，一致不知道具体原因，去网上搜索但是天下文章一大抄，你抄我来我抄他，也没找到原因，后来因为工作中也没遇到过，感觉应该是自己当时水平不行就忘了这件事，前几天 fatsjson 和 druid 的作者温少突然在一个群里面说有人通过阿里工单反馈这个问题，他给追踪了一下，找到了原因，原来还是还是有人遇到这个问题，今天记录一下，希望对遇到这个问题的小伙伴有帮助，报错的信息大概就是：

```

Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 20,820,001 milliseconds ago. The last packet sent successfully to the server was 20,820,002 milliseconds ago. is longer than the server configured value of &#8216;wait_timeout&#8217;. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property &#8216;autoReconnect=true&#8217; to avoid this problem.  
at sun.reflect.GeneratedConstructorAccessor29.newInstance(Unknown Source) 

…………

```

下面是温少分享的截图

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/11/MySQL_error-389x1024.png" alt="" width="389" height="1024" class="alignnone size-medium wp-image-430" /> 

感谢温少，感谢各种软件、框架的开源作者和企业，有了你们，程序员的生活的才更美好。

**2019-03-12 补充：**

据温少说，升级到 1.1.14 并且配置 keepAlive=true，问题可解决。另如果看报错日志，同时也会提示大家在 URL 中配置：autoReconnect=true