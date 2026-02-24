---
title: 关于synchronized用法的简单理解
author: Bridge Li
type: post
date: 2017-05-14T09:20:14+00:00

categories:
  - Java
tags:
  - synchronized
  - 多线程

---
synchronized 关键字既可以用于声明方法，也可以用于声明代码块，他们之间有什么区别呢？下面让我们逐一测试一下。  
先看以第一个例子：

```

package demo;

public class SynchronizedDemo1 {

public synchronized static void foo1() {  
}

public synchronized static void foo2() {  
}  
}

```

在这个例子中，foo1 和 foo2 是类的两个静态方法。在不同的线程中，这两个方法的调用时互斥的，不仅是他们之间，任何两个不同的线程的调用也互斥。下面看第二个例子：

```

package demo;

public class SynchronizedDemo2 {

public synchronized void foo3() {  
}

public synchronized void foo4() {  
}  
}

```

在这个例子中，foo3 和 foo4 是类的两个成员方法，在多线程环境中，调用同一个对象的 foo3 或者 foo4 是互斥的，与上一个例子的差别在于，这是针对同一个对象的多线程方法调用互斥。下面再看最后一个例子：

```

package demo;

public class SynchronizedDemo3 {

public void foo5() {  
synchronized (this) {

}  
}

public void foo6() {  
synchronized (SynchronizedDemo3.class) {

}  
}  
}

```

在这个例子中，synchronized 用来修饰代码块，需要注意的是：synchronized 后面会有一个参数，其实这个就是用于同步的锁所属的对象。在这个例子中 synchronized (this) 与 SynchronizedDemo2 中加 synchronized 的成员方法是互斥的，而 synchronized (SynchronizedDemo3.class) 与 SynchronizedDemo1 中加 synchronized 的静态方法是互斥的。synchronized 用于修饰代码块会更加灵活，因为除了前面的这个例子外，synchronized 后面的参数可以是任意对象。