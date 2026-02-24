---
title: ThreadLocal类之简单理解
author: Bridge Li
type: post
date: 2017-06-11T12:21:40+00:00

categories:
  - Java
tags:
  - ThreadLocal
  - 多线程
  - 线程安全

---
当年实习的时候，当时公司一个相当有经验的工程师zeak带我们，从他那第一次听说了ThreadLocal类，但由于自己基础薄弱，没有理解到底怎么回事，工作中也没有用过，就一直没有太放在心上，刚好这一段时间不太忙，仔细玩了一下，欢迎高手批评。  
ThreadLocal，线程本地变量。他为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。简单理解就是，对于非线程安全的变量在线程内部共享不用每次都new，是一种空间换时间的做法。ThreadLocal类提供的几个方法：

```

public T get() { }  
public void set(T value) { }  
public void remove() { }  
protected T initialValue() { }

```

看名字就知道这些方法是干嘛的了，下面我们来看一下ThreadLocal类是如何为每个线程创建一个变量的副本的。首先是get方法的实现：

```

public T get() {  
Thread t = Thread.currentThread();  
ThreadLocalMap map = getMap(t);  
if (map != null) {  
ThreadLocalMap.Entry e = map.getEntry(this);  
if (e != null)  
return (T)e.value;  
}  
return setInitialValue();  
}

```

先取得当前线程，然后通过getMap(t)方法获取到一个map，map的类型为ThreadLocalMap。然后接着下面获取到<key,value>键值对，注意这里获取键值对传进去的是 this，而不是当前线程t。如果获取成功，则返回value值。如果map为空，则调用setInitialValue方法返回value。那么getMap方法中又做了什么呢？

```

ThreadLocalMap getMap(Thread t) {  
return t.threadLocals;  
}

```

原来是返回当前线程t中的一个成员变量threadLocals，而threadLocals则是：

```

ThreadLocal.ThreadLocalMap threadLocals = null;

```

那就看看ThreadLocalMap的实现了：

```

static class ThreadLocalMap {

/**  
* The entries in this hash map extend WeakReference, using  
* its main ref field as the key (which is always a  
* ThreadLocal object). Note that null keys (i.e. entry.get()  
* == null) mean that the key is no longer referenced, so the  
* entry can be expunged from table. Such entries are referred to  
* as "stale entries" in the code that follows.  
*/  
static class Entry extends WeakReference<ThreadLocal> {  
/*\* The value associated with this ThreadLocal. \*/  
Object value;

Entry(ThreadLocal k, Object v) {  
super(k);  
value = v;  
}  
}

```

ThreadLocalMap的Entry继承了WeakReference，并且使用ThreadLocal作为键值。  
然后再继续看setInitialValue方法的具体实现：

```

private T setInitialValue() {  
T value = initialValue();  
Thread t = Thread.currentThread();  
ThreadLocalMap map = getMap(t);  
if (map != null)  
map.set(this, value);  
else  
createMap(t, value);  
return value;  
}

```

很容易了解，就是如果map不为空，就设置键值对，为空，再创建Map，看一下createMap的实现(顺便问一下ifelse为什么没有花括号啊)：

```

void createMap(Thread t, T firstValue) {  
t.threadLocals = new ThreadLocalMap(this, firstValue);  
}

```

至此，可能大部分朋友已经明白了ThreadLocal是如何为每个线程创建变量的副本的：  
首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。  
初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。  
然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

最后总结一下：

1. 实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的；  
2. 为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，因为每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal；  
3. 在进行get之前，必须先set，否则会报空指针异常；如果想在get之前不需要调用set就能正常访问的话，必须重写initialValue()方法（大家可以看一下源码为什么，很简单的）

参考资料：

1. http://www.cnblogs.com/dolphin0520/p/3920407.html  
2. jdk1.7.0_40源码