---
title: 你假笨JVM参数 – 003 CompileCommand
author: Bridge Li
type: post
date: 2017-11-25T10:15:16+00:00

categories:
  - 你假笨说JVM参数
tags:
  - CompileCommand
  - JVM

---
你假笨的第三次分享：

序号：003  
时间：2017-07-19  
参数：-XX:CompileCommand  
含义：  
Specifies a command to perform on a method.  
该参数用于定制编译需求，比如过滤某个方法不做JIT编译  
若未指定方法描述符，则对全部同名方法执行命令操作，具体如何指定见下文[举例]  
可使用星号通配符（*）指定类或方法，具体如何使用见下文[举例]  
该参数可多次指定，或使用 换行符（\n）分隔参数后的多个命令  
解析完该命令后，JIT编译器会读取.hotspot_compiler文件中的命令，该参数也可写在.hotspot_compiler文件中  
可使用-XX:CompileCommandFile指定.hotspot_compiler文件为其他文件  
用法：

```

-XX:CompileCommand=command,method[,option]  
命令：  
exclude，跳过编译指定的方法  
compileonly，只编译指定的方法  
inline/dontinline，设置是否内联指定方法  
print，打印生成的汇编代码  
break，JVM以debug模式运行时，在方法编译开始处设置断点  
quiet，不打印在此命令之后、通过-XX:CompileCommand指定的编译选项  
log，记录指定方法的编译日志，若未指定，则记录所有方法的编译日志  
其他命令，option，help

```

举例：  
1. 设置编译器跳过编译com.jvmpocket.Dummy类test方法的4种写法

```

-XX:CompileCommand=exclude,com/jvmpocket/Dummy.test  
-XX:CompileCommand=exclude,com/jvmpocket/Dummy::test  
-XX:CompileCommand=exclude,com.jvmpocket.Dummy::test  
-XX:CompileCommand="exclude com/jvmpocket/Dummy test"

```

2. 设置编译器只跳过编译java.lang.String类int indexOf(String)方法  
-XX:CompileCommand=&#8221;exclude,java/lang/String.indexOf,(Ljava/lang/String;)I&#8221;  
3. 设置编译器跳过编译所有类的indexOf方法  
-XX:CompileCommand=exclude,*.indexOf

小程序截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/11/CompileCommand_JVMPocket-522x1024.png" alt="" width="522" height="1024" class="alignnone size-medium wp-image-463" /> 

分享记录：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2017/11/CompileCommand-213x1024.png" alt="" width="213" height="1024" class="alignnone size-medium wp-image-462" />