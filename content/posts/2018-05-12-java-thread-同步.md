---
title: Java Thread 同步
author: Bridge Li
type: post
date: 2018-05-12T13:33:53+00:00

categories:
  - Java
tags:
  - join
  - thread
  - 同步
  - 多线程

---
之前遇到一个问题，就是如何让线程同步，由于自己多线程的东西实在不懂，所以不知道怎么办，但感觉应该是一个很简单的东西，所以就从网上搜一下资料，原来如此简单，直接调用 join 方法就好了。写篇博客记录一下 join 的使用方法。

1. 作用

Thread类中的join方法的主要作用就是同步，它可以使得线程之间的并行执行变为串行执行。具体看代码：

```

package cn.bridgeli.demo;

public class ThreadTest {

public static void main(String[] args) throws InterruptedException {  
ThreadJoinTest t1 = new ThreadJoinTest("bridgeli");  
ThreadJoinTest t2 = new ThreadJoinTest("liqiao");  
t1.start();  
/**  
* join的意思是使得放弃当前线程的执行，并返回对应的线程，例如下面代码的意思就是：  
* 程序在main线程中调用t1线程的join方法，则main线程放弃cpu控制权，并返回t1线程继续执行直到线程t1执行完毕  
* 所以结果是t1线程执行完后，才到主线程执行，相当于在main线程中同步t1线程，t1执行完了，main线程才有执行的机会  
*  
* join方法可以传递参数，join(10000)表示main线程会等待t1线程10毫秒，10毫秒过去后，  
* main线程和t1线程之间执行顺序由串行执行变为普通的并行执行  
*/  
t1.join(10000);  
t2.start();  
}

}

package cn.bridgeli.demo;

public class ThreadJoinTest extends Thread {  
public ThreadJoinTest(String name) {  
super(name);  
}

@Override  
public void run() {  
for (int i = 0; i < 100; i++) {  
System.out.println(this.getName() + ":" + i);  
try {  
Thread.sleep(1000);  
} catch (InterruptedException e) {  
e.printStackTrace();  
}  
}  
}  
}

```

上面注释也大概说明了 join 方法的作用：在 主线程 中调用了 t1 线程的 join() 方法时，表示只有当 t1 线程执行完毕时，主线程 才能继续执行，也就是开始执行 t2。注意，join 方法其实也可以传递一个参数给它的，表示：如果 主线程 在 t1 执行 1000 毫秒之后，继续执行，也就是开启 t2 线程。

2. join 与 start 调用顺序问题

如果我们把 t1 的 join 和 start 对调，我们会发现两个线程交替执行，也就是 join 方法必须在线程 start 方法调用之后调用才有意义。这个也很容易理解：如果一个线程都没有 start，那它也就无法同步了。

3. join 方法实现原理

```

/**  
* Waits for this thread to die.  
*  
* <p> An invocation of this method behaves in exactly the same  
* way as the invocation  
*  
* <blockquote>  
* {@linkplain #join(long) join}{@code (0)}  
* </blockquote>  
*  
* @throws InterruptedException  
* if any thread has interrupted the current thread. The  
* <i>interrupted status</i> of the current thread is  
* cleared when this exception is thrown.  
*/  
public final void join() throws InterruptedException {  
join(0);  
}

/**  
* Waits at most {@code millis} milliseconds for this thread to  
* die. A timeout of {@code 0} means to wait forever.  
*  
* <p> This implementation uses a loop of {@code this.wait} calls  
* conditioned on {@code this.isAlive}. As a thread terminates the  
* {@code this.notifyAll} method is invoked. It is recommended that  
* applications not use {@code wait}, {@code notify}, or  
* {@code notifyAll} on {@code Thread} instances.  
*  
* @param millis  
* the time to wait in milliseconds  
*  
* @throws IllegalArgumentException  
* if the value of {@code millis} is negative  
*  
* @throws InterruptedException  
* if any thread has interrupted the current thread. The  
* <i>interrupted status</i> of the current thread is  
* cleared when this exception is thrown.  
*/  
public final synchronized void join(long millis)  
throws InterruptedException {  
long base = System.currentTimeMillis();  
long now = 0;

if (millis < 0) {  
throw new IllegalArgumentException("timeout value is negative");  
}

if (millis == 0) {  
while (isAlive()) {  
wait(0);  
}  
} else {  
while (isAlive()) {  
long delay = millis &#8211; now;  
if (delay <= 0) {  
break;  
}  
wait(delay);  
now = System.currentTimeMillis() &#8211; base;  
}  
}  
}

```

从源码中可以看到：join 方法的原理就是调用相应线程的 wait 方法进行等待操作的，例如 主线程 中调用了 t1 线程的 join 方法，则相当于在 主线程 中调用了 t1 线程的 wait 方法，当 t1 线程执行完（或者到达等待时间），t1 线程会自动调用自身的 notifyAll 方法唤醒 主线程，从而达到同步的目的。

另外从源码中，我们还可以看到 join() 是调用了 join(0)，也就是 join(0) 的意思不是 主线程等待 t1 线程 0 秒，而是 主线程 等待 t1 线程无限时间，直到 t1 线程执行完毕，即 join(0) 等价于 join()。

参考资料：https://www.cnblogs.com/lcplcpjava/p/6896904.html