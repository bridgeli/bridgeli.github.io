---
title: Dubbo远程debug方法
author: Bridge Li
type: post
date: 2016-08-14T21:28:15+00:00

categories:
  - Java
tags:
  - debug
  - Dubbo
  - remote

---
公司项目的rpc服务基于阿里巴巴的dubbo架构，开发dubbo项目的时候测试只能跑junit test，但实际工作中由于很多时候junit test写的不全，出了问题只能再加日志分析原因（典型的没事找事型），这次和公司移动端的推送联调IM服务，发现他们已经把老夫之前听说的远程debug用在了实际工作中，刚好趁此机会实验了一把，以下是笔记，以待自己和需要的朋友参考。

1. dubbo服务的设置

我们自己观察dubbo的start.sh和start.bat这两个脚本会发现有如下两端代码

①. start.sh

```

JAVA_DEBUG_OPTS=""  
if [ "$1" = "debug" ]; then  
JAVA_DEBUG_OPTS=" -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n "  
fi

```

②. start.bat

```

if ""%1"" == ""debug"" goto debug  
if ""%1"" == ""jmx"" goto jmx

java -Xms64m -Xmx1024m -XX:MaxPermSize=64M -classpath ..\conf;%LIB_JARS% com.alibaba.dubbo.container.Main  
goto end

:debug  
java -Xms64m -Xmx1024m -XX:MaxPermSize=64M -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n -classpath ..\conf;%LIB_JARS% com.alibaba.dubbo.container.Main  
goto end

```

也就是说，脚本已经支持远程debug，只需要的在启动的时候传入一个参数 debug 即可，其余的几乎不用做任何修改

2. eclipse的设置

当我们把远程的服务以支持debug的模式启动之后，就需要把本地的项目也起来了，否则怎么debug呢，本地的设置其实非常简单，一张图搞定

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/08/remote_debug-300x168.png" alt="remote_debug-300x168" width="300" height="168" class="alignnone size-full wp-image-307" /> 

看了这张图，我相信不用我多说了，远程远程debug如此简单