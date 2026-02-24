---
title: Java集合类ArrayList删除特定元素
author: Bridge Li
type: post
date: 2017-05-28T04:25:52+00:00

categories:
  - Java
tags:
  - ConcurrentModificationException
  - foreach原理
  - List

---
前一段时间入职新公司，熟悉公司系统原有代码的时候，发现公司代码那个烂啊，系统能正常跑，都不能用侥幸来形容，就是创造了一个奇迹。因为里面不仅没有coding style，而且竟然有很明显的常识性错误。其中当我一眼指出最明显的早就应该出过问题的一个地方，项目组几乎所有成员，是的，几乎全部成员，都说这个还真不知道，涨知识了，那就是：Java集合类ArrayList删除特定元素。发现原来不是所有人都知道怎么做，这难道不是最基础的吗？唉，真不知道这些系统是怎么跑起来的。我们首先看两种错误的写法，第一种：

```

@Test  
public void testRemove1() {  
List<String> list = new ArrayList<String>() {  
private static final long serialVersionUID = 1L;

{  
add("cn");  
add("bridgeli");  
add("blog");  
}  
};  
for (int i = 0, len = list.size(); i < len; i++) {  
String str = list.get(i);  
if ("cn".equals(str)) {  
list.remove(str);  
}  
}  
}

```

这个写法如果你不知道错在哪，那你得真的好好补基础了。由于这个错误比较明显，所以有人搞了下面这种写法，也是我们公司的同事犯的一个错误：

```

@Test  
public void testRemove2() {  
List<String> list = new ArrayList<String>() {  
private static final long serialVersionUID = 1L;

{  
add("cn");  
add("bridgeli");  
add("blog");  
}  
};  
for (String str : list) {  
if ("cn".equals(str)) {  
list.remove(str);  
}  
}  
}

```

跑一下这个例子看看，把cn换成bridgeli试试，出乎不出乎你的意料？下面我们就来简单探究一下原因foreach的原理，其实特别简单：

Java.util.List实现了java.lang.Iterable接口。  
jdk api文档中是这样描述Iterable接口的：实现这个接口允许对象成为 &#8220;foreach&#8221; 语句的目标。看一下Iterable接口并没有发现啥特别之处，只是定义了一个迭代器而已。用javap看一下字节码，你就会发现：foreach语法最终被编译器转为了对Iterator.next()的调用。而作为使用者的我们，jdk并没用向我们暴露这些细节，我们甚至不需要知道Iterator的存在，怎么样编译器厉害吧？如果你还不相信，你可以用Iterator写了个遍历List的方法看一下字节码看看和foreach是不是一样。既然是调用了Iterator.next()，那这个方法有什么特殊吗？看看实现就知道了。

```

@SuppressWarnings("unchecked")  
public E next() {  
checkForComodification();  
int i = cursor;  
if (i >= size)  
throw new NoSuchElementException();  
Object[] elementData = ArrayList.this.elementData;  
if (i >= elementData.length)  
throw new ConcurrentModificationException();  
cursor = i + 1;  
return (E) elementData[lastRet = i];  
}

```

方法的第一行调用了checkForComodification()，而他的实现呢？

```

final void checkForComodification() {  
if (modCount != expectedModCount)  
throw new ConcurrentModificationException();  
}

```

到这应该很明显了吧，看list的remove()实现，每次remove()的时候modCount都会++，而expectedModCount只会在初始化的时候赋过一次值。然后他们的值就不一样了，然后你肯定知道了。看完了错误的写法，我们看看其他的最正确的写法：

```

@Test  
public void testRemove3() {  
List<String> list = new ArrayList<String>() {  
private static final long serialVersionUID = 1L;

{  
add("cn");  
add("bridgeli");  
add("blog");  
}  
};  
Iterator<String> iterator = list.iterator();  
while (iterator.hasNext()) {  
String str = iterator.next();  
if ("blog".equals(str)) {  
iterator.remove();  
}  
}  
}

```

这个为什么不会出错，我相信根据刚才的分析，大家一定可以找到答案。还有为什么说这个最正确呢？因为还有其他的写法，我个人认为那些写法实在太low就不展示了，所以大家今后有这种需求就用这种写法就好了。最后的最后推荐大家看一看阿里巴巴新出的Java开发手册，我相信这些肯定是阿里的同行踩过的坑的总结，里面不仅对coding style做了严格要求，而且还能避免很多常识性错误，而且有些还说明了原因，里面就有对这个问题的说明，所以看一下吧，虽然里面的部分观点你可能不认同，但我相信看完实践一下你会受益良多。