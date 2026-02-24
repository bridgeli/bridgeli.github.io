---
title: Maven Missing artifact解决之道
author: Bridge Li
type: post
date: 2014-09-03T13:22:21+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Maven
---
前几天没事用maven重构自己的微信公众平台开发的代码，当下载一个Jar包时，遇到一个问题：Missing artifact net.sf.json-lib:json-lib:jar:2.2.3，去仓库看，这个Jar包确实没下载下来，因为自己的maven是半路里出家的（自己完全在网上找的一些乱七八糟的资源自学的），所以不知道咋回事，于是就去网上找解决的办法，也许是没找对地方，死活就是解决不了（为避免误导大家，就不列举这些方法了），问同事怎么办，一同事说应该是你下载的时候的网断了之类的导致资源下载了一半，然后网再连上，就不接着下载了，感觉似乎挺有道理，删了还是不行，那是不是网站被和谐了，采用VPN结果还是不行，最后仔细观察，终于发现了一下端倪：

[<img loading="lazy" decoding="async" class="alignnone size-medium wp-image-47" src="https://www.bridgeli.cn/wp-content/uploads/2014/09/maven-300x65.png" alt="maven" width="300" height="65" />][1]

也就是说

```

<dependency>
  <groupId>net.sf.json-lib</groupId>
  <artifactId>json-lib</artifactId>
  <version>2.2.3</version>
</dependency>

```

对应多个Jar，maven不知道下载那个Jar，所以就报错了，正确的解决方法是加一个标签：<classifier>，即变成：

```

<dependency>
  <groupId>net.sf.json-lib</groupId>
  <artifactId>json-lib</artifactId>
  <version>2.2.3</version>
  <classifier>jdk13</classifier>
</dependency>

```

就没问题了。

 [1]: https://www.bridgeli.cn/wp-content/uploads/2014/09/maven.png