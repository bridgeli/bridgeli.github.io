---
title: Apache Commons Codec — 加密与编码
author: Bridge Li
type: post
date: 2019-09-30T06:59:19+00:00

categories:
  - Java
tags:
  - base64
  - MD5
---
明天就是十一假期了，公司也没多大事，刷知乎，看到有人吐槽曾经的一个合作伙伴连 md5 都写不对，告诉对方写错了，对方顾头不顾腚的修，还是没修对，然后测了一下自己写的想亏写对了，不然又遗留 bug 了，不过看下面评论，有人提到 Apache Commons Codec 里面都已经写好了，看了一下确实，反正也无聊，也写不了大文章，写着玩玩。

1. 先看自己手写的 md5 // 今后大家不要这么写了，太傻了

```

package cn.bridgeli.demo;

import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;

import java.security.MessageDigest;  
import java.security.NoSuchAlgorithmException;

/**  
* Created by bridgeli on 2019/7/12.  
*/  
public class EncryptUtils {

private static Logger logger = LoggerFactory.getLogger(EncryptUtils.class);

private EncryptUtils() {  
}

public static String getMD5(String content) {

if (null == content) {  
return "";  
}

MessageDigest messageDigest = null;  
try {  
messageDigest = MessageDigest.getInstance("md5");  
} catch (NoSuchAlgorithmException e) {  
logger.error("md5 error", e);  
}  
if (null == messageDigest) {  
return "";  
}  
messageDigest.update(content.getBytes());  
byte[] bytes = messageDigest.digest();  
StringBuilder stringBuilder = new StringBuilder();  
for (byte b : bytes) {  
String str = Integer.toHexString(b & 0xFF);  
if (str.length() == 1) {  
stringBuilder.append("0");  
}  
stringBuilder.append(str);  
}  
String result = stringBuilder.toString();

return result;  
}  
}

```

2. 借助别人写好的工具类实现

先引入依赖

```

<dependency>  
<artifactId>commons-codec</artifactId>  
<groupId>commons-codec</groupId>  
<version>1.9</version>  
</dependency>

```

具体实现

```

package cn.bridgeli.demo;

import org.apache.commons.codec.binary.Base64;  
import org.apache.commons.codec.binary.Hex;  
import org.apache.commons.codec.digest.DigestUtils;  
import org.junit.Test;  
import sun.misc.BASE64Decoder;  
import sun.misc.BASE64Encoder;

import java.io.IOException;  
import java.security.MessageDigest;  
import java.security.NoSuchAlgorithmException;

/**  
* Created by bridgeli on 2019/9/22.  
*/  
public class DigestUtilsTest {

@Test  
public void testMd5() {  
String string = DigestUtils.md5Hex("test");  
System.out.println(string);

MessageDigest md = null;  
try {  
md = MessageDigest.getInstance("MD5");  
} catch (NoSuchAlgorithmException e) {  
// 日志  
}  
byte[] md5Bytes = md.digest("test".getBytes());  
System.out.println(Hex.encodeHex(md5Bytes));

}

@Test  
public void testSHA256() {  
MessageDigest md = null;  
try {  
md = MessageDigest.getInstance("SHA-256");  
} catch (NoSuchAlgorithmException e) {  
// 日志  
}  
byte[] md5Bytes = md.digest("test".getBytes());  
System.out.println(Hex.encodeHex(md5Bytes));

System.out.println(DigestUtils.sha256Hex("test"));

}

@Test  
public void testBASE64() {  
BASE64Encoder encoder = new BASE64Encoder();  
String enStr = encoder.encode("test".getBytes());  
System.out.println(enStr);  
BASE64Decoder decoder = new BASE64Decoder();  
try {  
System.out.println(new String(decoder.decodeBuffer(enStr)));  
} catch (IOException e) {  
// 日志  
}

byte[] result = Base64.encodeBase64("test".getBytes());  
System.out.println(new String(result));  
System.out.println(new String(Base64.decodeBase64(result)));

}

}

```

最后想说的是，Apache commons 中有各种各样好用的工具类，没事的时候，翻翻就会有大收获，然后将来写代码的时候，不知道速度能提升多少倍

参考资料：https://www.jianshu.com/p/cf5d511d2db0