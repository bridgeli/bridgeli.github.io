---
title: Java中的split() replace() replaceFirst() replaceAll()四个函数分析
author: Bridge Li
type: post
date: 2014-10-20T02:34:49+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
---
前几天在公司分割一个很简单字符串，结果却怎么测都不对，最后查了一下资料，终于发现了端倪：  
split(regex);

replace(target, replacement);  
replace(oldChar, newChar); 

replaceFirst(regex, replacement);

replaceAll(regex, replacement)

仔细看一下，你会发现split()、replaceFirst()、replaceAll()的参数都是Regular Expression，也就是正则表达式，只有replace()的参数是字符或者字符串，由于这些参数类型的差异，很有将得不到预期的结果，下面是一些测试代码的例子，大家可以自己测一下

```

package cn.bridgeli.stringtest;
import org.junit.Test;

public class StringTest {

    @Test
    public void testSplit1() {
        String str = "111|222|333|444";
        String[] result = str.split("|");
        for (String string : result) {
            System.out.println(string);
        }

// String str = "111|222|333|444";  
// String[] result = str.split("\|");  
// for (String string : result) {  
// System.out.println(string);  
// }  
    }

    @Test
    public void testSplit2() {
        String str = "111,222,333,444";
        String[] result = str.split("\d");
        for (String string : result) {
            System.out.println(string);
        }

// String str = "111\222\333\444";  
// String[] result = str.split("\\");  
// for (String string : result) {  
// System.out.println(string);  
// }  
    }

    @Test
    public void testSplit3() {
        String str = "111222333444";
        String[] result = str.split("\");  
        for (String string : result) {
            System.out.println(string);
        }

// String str = "111\222\333\444";  
// String[] result = str.split("\\");  
// for (String string : result) {  
// System.out.println(string);  
// }  
    }

    @Test
    public void testReplaceAll() {
        String str = "111,222,333,444";
        String result = str.replaceAll(",", "$");
        System.out.println(result);

// String str = "111,222,333,444";  
// String result = str.replaceAll(",", "\$");  
// System.out.println(result);  
    }

}

```