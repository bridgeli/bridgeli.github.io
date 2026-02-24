---
title: 再谈 ThreadLocal
author: Bridge Li
type: post
date: 2021-04-22T09:49:30+00:00

categories:
  - Java
tags:
  - ThreadLocal
---
几年前我曾经写过两篇关于 ThreadLocal 的文章，分别是[ThreadLocal类之简单理解][1]和[ThreadLocal类之简单应用示例][2]，不过限于当时的水平，有些问题并没有说的很明白，所以今天再写一篇文章，重新说说这个类。

我们首先看一个例子：

```

package cn.bridgeli.demo;

/**  
* @author BridgeLi  
* @date 2021/4/21 11:02  
*/  
public class User {

String name = "Denny";

}

```

然后我们有一个操作：

```

package cn.bridgeli.demo;

import org.junit.Test;

/**  
* @author BridgeLi  
* @date 2021/4/21 10:28  
*/  
public class ThreadTest {

private User user = new User();

@Test  
public void testThreadLocal() {  
new Thread(() -> {  
try {  
Thread.sleep(1000);  
} catch (InterruptedException e) {  
e.printStackTrace();  
}  
System.out.println(user.name);  
}).start();

new Thread(() -> user.name = "BridgeLi").start();

try {  
Thread.sleep(2000);  
} catch (InterruptedException e) {  
e.printStackTrace();  
}  
}  
}

```

这个时候我们就知道一定会有线程安全问题，所以我们怎么解决这个问题呢？就是 ThreadLocal，请看下面：

```

package cn.bridgeli.demo.reference;

import org.junit.Test;

/**  
* @author BridgeLi  
* @date 2021/4/21 10:28  
*/  
public class ThreadLocalTest {

private static ThreadLocal<User> threadLocal = new ThreadLocal<>();

@Test  
public void testThreadLocal() {  
new Thread(() -> {  
try {  
Thread.sleep(1000);  
} catch (InterruptedException e) {  
e.printStackTrace();  
}  
System.out.println(threadLocal.get());  
}).start();

new Thread(() -> threadLocal.set(new User())).start();

try {  
Thread.sleep(2000);  
} catch (InterruptedException e) {  
e.printStackTrace();  
}  
}  
}

```

这个时候我们发现，一个线程里面放入的对象，我们在另一个线程拿不到，就这样解决了线程安全问题，那么这个又是如何做到的呢？我们通过源码分析，我们首先看 set 方法：

```

public void set(T value) {  
Thread t = Thread.currentThread();  
ThreadLocalMap map = getMap(t);  
if (map != null)  
map.set(this, value);  
else  
createMap(t, value);  
}

```

第一步先获取 当前线程，第二步获取当前线程的一个属性：threadLocals，类型是 ThreadLocalMap，这一步也就是说，在不同的线程里面，获取到的 t 对象肯定不是同一个，那么我们的 map 对象当然也就不是同一个了，所以下一步 set 的时候，不同的线程，也就 set 到了不同的对象里面去了，那么我们 get 的时候：

```

public T get() {  
Thread t = Thread.currentThread();  
ThreadLocalMap map = getMap(t);  
if (map != null) {  
ThreadLocalMap.Entry e = map.getEntry(this);  
if (e != null) {  
@SuppressWarnings("unchecked")  
T result = (T)e.value;  
return result;  
}  
}  
return setInitialValue();  
}

```

前面两步一样，都是获取当前线程，然后取当前线程上的 ThreadLocalMap，然后从 ThreadLocalMap 中获取存储的值，所以我们就轻而易举的理解了，上面的例子，为什么一个线程存，另一个线程取不到。那么这篇文章就到此结束了吗？当然没有。下面我们接着说 set 方法，我们先说第一个问题：

1. set 的时候的 key

我们很明显看到是 this，那么 this 是什么？就是 ThreadLocal 对象，到我们上面那个具体的例子就是我们定义的那个属性：threadLocal，那么这个时候很多人就有了一个问题：一个 ThreadLocal 只能存一个对象吗？答案肯定是：是的，因为你在存第二个对象的时候，由于 this 是同一个，所以一定会被覆盖掉，所以一个 ThreadLocal 对象只能存储一个对象，那么很多人很快就有了第二个疑问：那么我有两个对象要存怎么办？答案也很简单，再定义一个属性：threadLocal2 喽，不然还能咋办，那么此时两个对象是怎么存储的呢？我们可以想象的到，他们都在当前线程的一个 ThreadLocalMap 中存储，但是他们的 key 是不同的，分别对应不同的 ThreadLocal 对象，所以相安无事。下面我们接着研究第二个问题：

2. ThreadLocal 的内存泄漏问题

之前看到过一些文章，ThreadLocal 有内存泄漏问题，也有人说没有，那么 ThreadLocal 是否真的有内存泄漏问题？如果没有，为什么？如果有，同样为什么？看 ThreadLocal 有没有内存泄漏问题，这个时候我们就需要研究 ThreadLocalMap 这个对象了，所以我们接着看一个这个对象有何特点。

我们通过源码可以看到，我们初始化线程的时候有一行代码：

```

ThreadLocal.ThreadLocalMap threadLocals = null; 

```

也就是说 ThreadLocalMap 的初始值是 null，所以我们 set 的时候，getMap(t) 得到的一定是 null，代码进入到 else 取执行：createMap(t, value)，这个方法很简单就一行实现：

```

t.threadLocals = new ThreadLocalMap(this, firstValue);

```

也就是 new 了一个 ThreadLocalMap 对象，赋值给了当前线程的 threadLocals 属性，所以我们要看 ThreadLocalMap 的构造方法：

```

ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {  
table = new Entry[INITIAL_CAPACITY];  
int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY &#8211; 1);  
table[i] = new Entry(firstKey, firstValue);  
size = 1;  
setThreshold(INITIAL_CAPACITY);  
}

```

也很简单 new 了一个 Entry 对象，这个我们都很熟了，和 HashMap 很类似，那么他和 HashMap 的 Entry 是不是一样的呢？我们接着看一下：

```

static class Entry extends WeakReference<ThreadLocal<?>> {  
/*\* The value associated with this ThreadLocal. \*/  
Object value;

Entry(ThreadLocal<?> k, Object v) {  
super(k);  
value = v;  
}  
}

```

我们可以看到这个 Entry 有点不太一样，他继承了 WeakReference 对象，而且他的构造方法第一行调用了 super(k)，所以这个时候就很明显了，这个 k 就是我们定义的 ThreadLocal 属性，而这个时候，又调用了 WeakReference 的构造方法，我之前曾写过一篇文章：[Java 的引用类型和使用场景][3]，里面讲到了弱引用，也就是 WeakReference，他有什么特点？遇到 GC，不管三七二十一，立马就被回收了，所以就是基于此，有人说：ThreadLocal 没有了内存泄漏问题，因为 GC 之后，ThreadLocal 对象就会被回收，这个说法看似很正确，但是其实真的是这样的吗？我们再仔细看一下 Entry 的构造方法，他的 key 是 WeakReference，但是他的 value 呢？那可是一个扎扎实实的强引用，所以 GC 之后 key 被回收了，没有问题，value 咋办？还在占用内存，只是访问不到了而已？这块内存一直不会被释放，那么这不是内存泄漏是什么？所以 ThreadLocal 如果使用不当依然会有内存泄漏问题，使用的时候还是要小心滴。

看完上面这段话，有人发现，哎，不对啊，你不说我还用的好好的，你一说，把我说糊涂了，糊涂在哪呢？就在 WeakReference 这，WeakReference 是弱引用没错的，k 也确实是 ThreadLocal 对象，按照弱引用的特点，一 GC 不管三七二十一就把这个 key 回收了，在我们的系统中 GC 是随时都有可能发生的，而且不是受我们控制的，那么我们怎么取出的我们放进去的对象的呢？GC 之后不应该取不到了吗？其实这个问题，还是有些人没想清楚，我们再回过头来看一下我们最开始定义 ThreadLocal 的时候的代码：

```

private static ThreadLocal<User> threadLocal = new ThreadLocal<>();

```

这是一个啥？一个标标准准的强引用，所以我们不用担心 GC 之后，ThreadLocal 对象被回收，取不到我们存入的对象的问题，但是如果我们在使用的过程中把他赋值为 null，那么下次 GC 的时候，一定会被回收掉的，这是毫无疑问的。所以这个时候，又有同学有疑问了，那么既然如此：k 为什么还要定义成 WeakReference 类型的呢？这就是一个说法的问题，如果不定义成 WeakReference 类型的，那么一个内存就会有两个强引用存在，一个是我们定义的 threadLocal，另一个是这个 k，所以当我们把 threadLocal 赋值成 null 之后，我们会发现，无论如何我们也回收不掉这块内存，那么这不就是说 ThreadLocal 对象存在内存泄漏问题吗？

所以综上，我们可以得出这样一个结论：由于 WeakReference 的存在，ThreadLocal 对象本身没有内存泄漏问题，但是如果使用不当，我们存入的 value 会有内存泄漏问题。

看到这，又有同学有疑问了，那么 value 为啥，不也定义成 WeakReference 类型呢？那么不就 k 和 value 都没有内存泄漏问题了？其实原因也很简单，我们的 k 定义成 WeakReference 类型，不怕被回收，是因为有我们自己定义的 threadLocal 属性，这个强引用在，那么 value 呢？他没有啊，所以如果 value 也是 WeakReference 类型，那么一遇到 GC 完蛋了，我们放入的对象没了，所以 value 不能定义成 WeakReference 类型，那么既然如此，我们又该如何避免 value 的内存泄漏问题呢？答案很简单，使用完 ThreadLocal 之后，记得调用一下 remove 方法即可，具体就不再多说了。最后上一张 ThreadLocal 的内存结构图：

<img loading="lazy" decoding="async" src="http://www.bridgeli.cn/wp-content/uploads/2021/04/threadlocal-300x172.png" alt="" width="300" height="172" class="alignnone size-medium wp-image-704" />

 [1]: http://www.bridgeli.cn/archives/366
 [2]: http://www.bridgeli.cn/archives/367
 [3]: http://www.bridgeli.cn/archives/697