---
title: '神奇的 (a == (Integer) 1 && a == (Integer) 2 && a == (Integer) 3) = true'
author: Bridge Li
type: post
date: 2021-10-31T03:24:36+00:00

categories:
  - Java
tags:
  - 反射
  - 源码

---
前一段时间看了一篇文章 (a == (Integer) 1 && a == (Integer) 2 && a == (Integer) 3) 是否可以为 true，当时第一反应怎么可能，谁知道再往下看，作者竟然给出来如下代码，一运行神奇的事出现了，真的为 true，代码如下：

```

package cn.bridgeli.demo;

import java.lang.reflect.Field;

public class Magic {

public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {  
Class cache = Integer.class.getDeclaredClasses()[0];  
Field c = cache.getDeclaredField("cache");  
c.setAccessible(true);  
Integer[] array = (Integer[]) c.get(cache);  
// array[129] is 1  
array[130] = array[129];  
// Set 2 to be 1  
array[131] = array[129];  
// Set 3 to be 1  
Integer a = 1;  
if (a == (Integer) 1 && a == (Integer) 2 && a == (Integer) 3) {  
System.out.println(true);  
} else {  
System.out.println(false);  
}

}  
}

```

因为作者没有给出解释，所以就研究了一番，发现需要基础非常扎实才能写出这段代码，这段代码之所以为 true，要理解如下几个问题：

1. (Integer) 1 做了什么？

很多人可能感觉这还不简单，自动装箱呗，对象类型的 1，还有啥？其实这行代码经过编译之后，调用的是 Integer 的 valueOf 方法，实现如下：

```

public static Integer valueOf(int i) {  
if (i >= IntegerCache.low && i <= IntegerCache.high)  
return IntegerCache.cache[i + (-IntegerCache.low)];  
return new Integer(i);  
}

```

重要的是在那个 if 里面，取了缓存中的一个数，也就是说，某些数会提前在缓存中缓存好的，说一个题外话，这些数是哪些呢？IntegerCache.low 和 IntegerCache.high 之间，我们读一下源码，会发现 IntegerCache.low 固定是 -128，而 IntegerCache.high 默认是 127，当然也可以通过设置参数 java.lang.Integer.IntegerCache.high 调整，这里我们先不考虑调整的问题，就考虑 默认值。也就是从 -128 到 127 这 256 个数是提前缓存在内存中的，这个范围之外的数据，需要重新 new 对象，所以这也解释了为什么 (Integer) 127 == (Integer) 127 为 true，(Integer) 128 == (Integer) 128 为 false。好了题外话结束，继续我们的正题，我们知道了从 -128 到 127 这 256 个数是提前缓存到内存中的就好办了，那么他是缓存在哪呢？很明显可以看到是 IntegerCache 类的一个 cache 数组中，所以我们读一下代码就会发现这个数组从 0 到 255 刚好放着 从 -128 到 127 之间的这 256 个数字。

2. 缓存数组中的存放数据顺序

这个比较简单，就不多说了，数组下标肯定是从 0 开始，数据的下限从 -128 开始，所以数组位置为 0 的地方放着 -128，然后往后排就好了，位置 1 是 -127，位置128 是 0，位置255 是 127，如果设置了 java.lang.Integer.IntegerCache.high 为 1000，那就接着往后排就好了，位置 256 是 128，位置 1128 是 1000。同理我们也可以得出结论：数据 -128 在数组位置为 0 的地方，数据 0 在数组位置为 128 的地方等等。所以最终的结论就是，如果我们写一个 Integer a = 1 或者 (Integer) 1，根据 return IntegerCache.cache[i + (-IntegerCache.low)]，那么我们取的就是数组 cache 位置为 129 的那个位置的数据，因为原本那个位置的数据的值是 1。

3. 反射

通过上两步，我们知道了哪些数据被缓存了，也知道了被缓存到了哪个地方，同时也知道了，我们要取一个数的时候，取的是这个数组的哪个位置的数据，所以我们是否可以修改缓存好的某个位置的数据，从而使我们取数的时候，取得数不是原本的数，而是我们修改后的数呢？答案是：可以，因为我们有强大的，无所不能的反射。使用反射，我们通过Integer 对象的 Class 对象拿到 Integer 的内部类 IntegerCache 的 Class 对象，然后从 IntegerCache 的 Class 对象中获取数组 cache 属性，所以接下来就是修改数组 cache 中的数据，根据上一步我们可以很轻松的就能知道数组的第 129 位置的元素的值是 1，所以我们可以直接复制 array[130] = array[129]; 这样 130 位置的数据本来是 2 就被我们改成了 1。那么如果我们直接 array[130] = 1 行不行呢？其实是不行的，因为这个 cache 数组也是 Integer 类型的，所以也会存在自动装箱的问题。

4. 见证奇迹的时刻

现在我们再看这几行代码，就会发现首先 Integer a = 1，取的是 cache 数组中值位 129 那个位置的数据，我们没有修改这个位置的数据，所以 a 的值就是 1，而那个 if 条件，第一个 (Integer) 1 就不说了，我们看 (Integer) 2，取的是数组位置为 130 的数据，刚好这个地方被我们通过 array[130] = array[129] 赋值语句给修改了，也就是原本是 2 被我们修改成了 1，同理 (Integer) 3) 取的是 数组 cache 位置 131 的数据也被我们修改成了 1，所以此时奇迹就发生了：(a == (Integer) 1 && a == (Integer) 2 && a == (Integer) 3) = true。

小结：

其实这道题也挺无聊的，完全就是一个炫技的题，工作中一点用途都没有，但是通过这道题：

1. 可以看出我们的发散思维能力，是否一开始就认为不可能？是真的不可能，还是我们对自己使用的编程语言不熟悉，懒于思考；  
2. 也能看出我们对源码的熟悉程度，甚至对编辑器的理解，是否知道你写下的这行代码被编译成了什么？是否知道 Integer 类底层是什么实现的？是否知道自动装箱拆箱是怎么做的？源码也是面试考察中的一个重点。  
3. 最后也能看出我们对反射是否熟悉，Java 因为有了反射，所以才出现了 Spring 这个强大的生态型的框架，大大拓展了 Java 语言的生命力，所以反射也是我们 Java 程序员必须掌握的一个东西，也是面试中必考察的一个重点。