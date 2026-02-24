---
title: AntPathMatcher 路径匹配
author: Bridge Li
type: post
date: 2022-03-27T03:09:39+00:00

categories:
  - Java
tags:
  - spring mvc
---
公司项目使用 AntPathMatcher 路径匹配是否登陆，之前没有接触过，刚好趁这次机会学习了一番。

一、基本规则

1、? 匹配一个字符（除过操作系统默认的文件分隔符）  
2、* 匹配0个或多个字符  
3、** 匹配0个或多个目录  
4、{spring:[a-z]+} 将正则表达式 [a-z]+ 匹配到的值，赋值给名为 spring 的路径变量

PS：必须是完全匹配才行，在 SpringMVC 中只有完全匹配才会进入 controller 层的方法

二、注意事项：

1、匹配文件路径，需要匹配某目录下及其各级子目录下所有的文件，使用 /*\*/\* 而非 \*.\*，因为有的文件不一定含有文件后缀  
2、匹配文件路径，使用 AntPathMatcher 创建一个对象时，需要注意 AntPathMatcher 也有有参构造，传递路径分隔符参数 pathSeparator，对于文件路径的匹配来说，可以根据不同的操作系统来传递各自的文件分隔符，以此防止匹配文件路径错误  
3、最长匹配规则（has more characters），即越精确的模式越会被优先匹配到。例如，URL请求 /app/dir/file.jsp，现在存在两个路径匹配模式 /*\*/\*.jsp 和 /app/dir/\*.jsp，那么会根据模式 /app/dir/\*.jsp 来匹配

三、实例

可以参考若依框架：com.ruoyi.gateway.filter.AuthFilter 和 com.ruoyi.gateway.filter.XssFilter