---
title: NullPointerException in Java with no StackTrace
author: Bridge Li
type: post
date: 2018-01-07T10:39:30+00:00

categories:
  - Java
tags:
  - OmitStackTraceInFastThrow
  - 堆栈
  - 异常

---
这周一个项目遇到一个问题，同事查看日志发现抛出：NullPointerException，却没有堆栈信息，然后同事感觉很奇怪，因为打日志的方法，打印的确实是：e，而不是很多人不明所以的打印的：e.getMessage()。然后我看了一下想起来我看过某本书上说过的，JIT 优化。当某个异常抛出很多次之后，由于 Java 虚拟机 JIT 优化，会省略堆栈信息。往上面翻日志肯定可以会找到报错的地方，当然会出现报错的信息太多，比较难翻。写这篇文章的本来想找找那本书，参考一下的，结果忘了是那本书了，一时没找到，不过这个问题虽然不是非常常见，但是网上还是有很多说明的，所以就简单说说 JVM 有一个参数：OmitStackTraceInFastThrow 来控制是否开启此优化。关于此参数的简单说明：

JVM参数-XX:-OmitStackTraceInFastThrow参数可以关掉JVM对堆栈信息的优化。如果设置了这个参数，那么异常堆栈就能完整输出了。

“在服务器中的VM编译器现在提供准确的所有的“冷”内置异常堆栈回溯功能。为了性能考虑，当这些异常被抛出很多次时，这个方法会被重新编译，此后编译器将使用一种更快的抛出异常的方式，即抛出预先分配好的不带堆栈信息的异常。要完全关闭掉这种预分配的异常，就需要使用-XX:-OmitStackTraceInFastThrow参数。”

相关stackoverflow讨论：https://stackoverflow.com/questions/2411487/nullpointerexception-in-java-with-no-stacktrace

笨神小程序 JVMPocket 对此参数的解释截图：

<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2018/01/OmitStackTraceInFastThrow-305x1024.png" alt="" width="305" height="1024" class="alignnone size-medium wp-image-510" /> 

另，看到网上有部分人建议关闭此参数，个人是建议的：

1. 由于Java 在虚拟机中运行，天生是比 c 要慢的，为了优化此问题，那些 JVM 大神们搞出了 JIT 优化，对性能有了大幅提升。  
2. 异常信息使我们应该特别关注了，当出现了异常应该尽快发现问题解决问题，而不是这个异常发生很多次了但是一直不知道。  
3. 打详细堆栈信息是很影响性能的，如果这个异常出现很多次了，没必要每次都打出来异常，看一个地方就知道了，而不是让他一直打，影响性能。

那有人说了当遇到此问题时，堆栈信息看不到，日志又比较难翻怎么办？我们好像也不能动态的添加 JVM 的启动参数，所以保险起见还是关闭此参数。个人认为：这个异常这么常出现，而且是 JIT 的优化导致看不到堆栈信息了，可以找一台机器重启一下就好了，没必要因小失大，我们从笨神的小程序也可以看到 JVM 默认是开启此参数的，开启肯定是有其道理的。