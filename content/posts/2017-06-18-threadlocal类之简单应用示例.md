---
title: ThreadLocal类之简单应用示例
author: Bridge Li
type: post
date: 2017-06-18T12:50:29+00:00

categories:
  - Java
tags:
  - SimpleDateFormat
  - 时间
  - 格式换

---
在日常开发的系统中，日期处理是非常非常用的一个功能，处理的日期的时候就需要用到SimpleDateFormat对象，但是我们都知道SimpleDateFormat本身不是线程安全的（如果不知道的请看源码），所以就需要频繁创建SimpleDateFormat这个对象。但是我们知道创建这个对象本身不仅是很费时的，而且创建的这些对象存活期很短，导致内存中大量这样的对象需要被GC，所以我们自然而然的想到使用ThreadLocal来给每个线程缓存一个SimpleDateFormat实例，提高性能。下面是一个具体的实现的小例子，其实不仅针对SimpleDateFormat对象，对于数据库连接等等都可以这么使用。

```

package cn.bridgeli.demo;

import java.text.DateFormat;  
import java.text.SimpleDateFormat;  
import java.util.HashMap;  
import java.util.Map;

public class DateFormatFactory {

private static final Map<DatePatternEnum, ThreadLocal<DateFormat>> pattern2ThreadLocal;

static {  
DatePatternEnum[] patterns = DatePatternEnum.values();  
int len = patterns.length;  
pattern2ThreadLocal = new HashMap<DatePatternEnum, ThreadLocal<DateFormat>>(len);

for (int i = 0; i < len; i++) {  
DatePatternEnum datePatternEnum = patterns[i];  
final String pattern = datePatternEnum.pattern;

pattern2ThreadLocal.put(datePatternEnum, new ThreadLocal<DateFormat>() {  
@Override  
protected DateFormat initialValue() {  
return new SimpleDateFormat(pattern);  
}  
});  
}  
}

// 获取DateFormat  
public static DateFormat getDateFormat(DatePatternEnum patternEnum) {  
ThreadLocal<DateFormat> threadDateFormat = pattern2ThreadLocal.get(patternEnum);  
// 不需要判断threadDateFormat是否为空  
return threadDateFormat.get();  
}  
}

```

对应的时间枚举类（如果还有其他格式的时间需要处理，可以直接在这个类里面添加即可）：

```

package cn.bridgeli.demo;

public enum DatePatternEnum {

TimePattern("yyyy-MM-dd HH:mm:ss"),  
DatePattern("yyyy-MM-dd");

public String pattern;

private DatePatternEnum(String pattern) {  
this.pattern = pattern;  
}  
}  
```

这样我们就可以每次调用DateFormatFactory.getDateFormat获取到对应的时间格式化类了。之前我们提到使用ThreadLocal同时可以避免参数传递。假如某个类的某个方法要调用到其他类的方法，而且方法内需要使用时间格式化类。按照正常情况下我们把该时间格式化类作为参数进行传递，但如果有了ThreadLocal这个类，我们可以不需要作为参数传递了，直接在方法类通过ThreadLocal得到时间格式化类。

参考资料：

1. https://segmentfault.com/a/1190000000537475