---
title: 关于 alibaba fastjson 的两个小知识点
author: Bridge Li
type: post
date: 2019-08-25T08:22:47+00:00

categories:
  - Java
tags:
  - fastjson
  - gson
  - json

---
1. json 转 JavaBean 大小写不敏感

在工作中，我个人经常使用的 json 的工具类是 Google 的 gson，前几天做一个需求的也自然而然的使用这个，但是在和其他部门联调的时候，发现他的属性全是小写，而不是刚开始约定的小驼峰，所以导致他传过来的字符串，我这边转不成 Java bean，在和他讨论的时候，他说他一直就这么写，而且别人使用的时候是没有问题的，然后就找到其他的使用的同学，他竟然说他没在意过这个问题，线上测试了一下，确实没有问题，然后就看一下怎么实现的，然后突然发现，他使用的是 alibaba 的 fastjson 转 Java bean，然后测试了一下，发现 alibaba fastjson 转 Java bean 竟然是大小写不敏感的，特此记录一下，测试代码入校:

```

<dependency>  
<groupId>com.alibaba</groupId>  
<artifactId>fastjson</artifactId>  
<version>1.2.37</version>  
</dependency>

package cn.bridgeli.demo;

import com.alibaba.fastjson.JSON;  
import org.junit.Test;

import java.io.Serializable;  
import java.util.ArrayList;  
import java.util.List;

public class FastJsonTest {

@Test  
public void testFastJsonParseObject() {

String jsonStr = "{\"USERNAME\":\"BridgeLi\"}";

User user = JSON.parseObject(jsonStr, User.class);  
System.out.println(user.getUsername());

}  
}

class User implements Serializable {

private static final long serialVersionUID = 4202834388700617773L;

private String username;

public String getUsername() {  
return username;  
}

public void setUsername(String username) {  
this.username = username;  
}  
}

```

我们看到测试的例子中 Java bean 的属性是小写的，json 字符串中的属性是全大写的，但转换一点问题都没有。

2. 循环依赖

先看例子：

```

package cn.bridgeli.demo;

import com.alibaba.fastjson.JSON;  
import org.junit.Test;

import java.io.Serializable;  
import java.util.ArrayList;  
import java.util.List;

public class FastJsonTest {

@Test  
public void testFastJsonToString() {

List<User> users = new ArrayList<>();  
User user = new User();  
user.setUsername("BridgeLi");

users.add(user);  
users.add(user);  
String jsonStr = JSON.toJSONString(users);  
System.out.println(jsonStr);

}

}

class User implements Serializable {  
private static final long serialVersionUID = 4202834388700617773L;

private String username;

public String getUsername() {  
return username;  
}

public void setUsername(String username) {  
this.username = username;  
}  
}

```

仔细看输入有一个：{&#8220;$ref&#8221;:&#8221;$[0]&#8221;}，这个是因为：fastjson支持循环引用，并且是缺省打开的。具体可以参考 wiki：

https://github.com/alibaba/fastjson/wiki/%E5%BE%AA%E7%8E%AF%E5%BC%95%E7%94%A8

个人感觉这个属性，其实挺有意思的，不过不支持的话，不知道是不是作者担心出现循环依赖，然后导致内存溢出的问题。最后需要说明的是 Google 的 gson 中这个循环依赖默认就是关闭的，所以不会出现序列化后的 JSON 传输到浏览器或者其他语言中解析不了的问题，大家感兴趣的话可以测试一下是否会出现内存溢出的问题。