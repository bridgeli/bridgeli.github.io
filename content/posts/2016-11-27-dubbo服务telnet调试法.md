---
title: Dubbo服务telnet调试法
author: Bridge Li
type: post
date: 2016-11-27T13:12:26+00:00

categories:
  - Java
tags:
  - debug
  - Dubbo
  - Telnet

---
公司的RPC的服务使用的是阿里巴巴的dubbo，老夫之前曾经写过一篇如何在测试环境[远程调试dubbo服务][1]，详情请参考这篇，但一直对如何调试线上dubbo服务不得法，不得已每次都需要写一个web服务调一下看数据，前一段时间经新来的一个同事提示可以使用Telnet调试，网上搜了一下资料，发现真的很爽，以下是学习笔记。  
需要说明的是：Dubbo2.0.5以上版本服务提供端口支持telnet命令，不过应该没有公司使用2.0.5以下版本吧。

1. 进入调试模式

```

telnet localhost 20880

```

即：telnet + ip + 端口，这个不用解释，使用dubbo的肯定都知道

2. ls

使用上一个命令之后，敲一下回车，就进入dubbo的telnet调试服务了，然后就可以使用ls命令了，这个命令有几个用法：

①. 显示服务列表

```

ls

```

②. 显示服务详细信息列表

```

ls -l

```

③. 显示服务的方法列表

```

ls XxxService

```

④. 显示服务的方法详细信息列表

```

ls -l XxxService

```

3. ps

这个命令主要是看连接信息，也有如下几个用法：

①. 显示服务端口列表

```

ps

```

②. 显示服务地址列表

```

ps -l

```

③. 显示端口上的连接信息

```

ps 20880

```

④, 显示端口上的连接详细信息

```

ps -l 20880

```

4. cd

这个缺省服务，主要有以下两个用法：

①. 改变缺省服务，当设置了缺省服务，凡是需要输入服务名作为参数的命令，都可以省略服务参数

```

cd XxxService

```

②. 取消缺省服务

```

cd /

```

5. pwd

显示当前缺省服务

6. trace

这个命令顾名思义：跟踪，具体有以下用法：

①. 跟踪1次服务任意方法的调用情况

```

trace XxxService

```

②. 跟踪10次服务任意方法的调用情况

```

trace XxxService 10

```

③. 跟踪1次服务方法的调用情况

```

trace XxxService xxxMethod

```

④. 跟踪1次服务方法的调用情况

```

trace XxxService xxxMethod 10

```

7. count

统计命令，有如下几个用法：

①. 统计1次服务任意方法的调用情况

```

count XxxService

```

②. 统计10次服务任意方法的调用情况

```

count XxxService 10

```

③. 统计1次服务方法的调用情况

```

count XxxService xxxMethod

```

④. 统计10次服务方法的调用情况 

```

count XxxService xxxMethod 10

```

8. invoke

这个命令，算是最核心的命令，调试的时候主要就是使用这个命令调试某个服务的某个方法，用法如下：

①. 调用服务的方法。

```

invoke XxxService.xxxMethod({"prop": "value"})

```

②. 调用服务的方法(自动查找包含此方法的服务)

```

invoke xxxMethod({"prop": "value"})

```

需要说明的是：

①. 如果调试的方法的参数是个对象，那么调试的时候，参数必须是json字符串（这个不难理解，dubbo调用的时候需要对象实现序列化接口，使参数序列化和反序列化）  
②. 如果是基本类型，例如int，则应该直接：

```

invoke xxxMethod(1)

```

9. status

这个我没有用过，大家可以自己试试，用法如下：

①. 显示汇总状态，该状态将汇总所有资源的状态，当全部OK时则显示OK，只要有一个ERROR则显示ERROR，只要有一个WARN则显示WARN。

```

status

```

②. 显示状态列表

```

status -l

```

10. log

这个我也没有用过，大家可以自己试试，需要说明的是：2.0.6以上版本支持才支持这个命令，我相信所有公司用的版本肯定都支持这个命令，用户如下：

①. 修改dubbo logger的日志级别

```

log debug 

```

②. 查看file logger的最后100字符的日志

```

log 100

```

11. help

这个不用解释，用户如下：

①. 显示telnet命帮助信息

```

help

```

②. 显示xxx命令的详细帮助信息。

```

help xxx

```

12. clear

这个也不用解释，就是清屏而已，用法如下：

①. 清除屏幕上的内容。

```

clear

```

②. 清除屏幕上的指定行数的内容  
```

clear 100

```

13. exit

退出当前telnet命令行

自从加上[Dubbo远程debug方法][1]天终于亮了

 [1]: https://www.bridgeli.cn/archives/306 "Dubbo远程debug方法"