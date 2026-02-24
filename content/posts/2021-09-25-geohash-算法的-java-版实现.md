---
title: GeoHash 算法的 Java 版实现
author: Bridge Li
type: post
date: 2021-09-25T06:02:54+00:00

categories:
  - Java
tags:
  - GeoHash
  - LBS

---
之前曾经做过一个类 LBS 的小需求，当时是用 redis 做的，就是[这篇文章][1]，其实 GeoHash 算法，我们也可以自己实现，具体如下：

```

package cn.bridgeli.demo;

import java.util.ArrayList;  
import java.util.BitSet;  
import java.util.HashMap;

public class GeoHash {  
public static final double MINLAT = -90;  
public static final double MAXLAT = 90;  
public static final double MINLNG = -180;  
public static final double MAXLNG = 180;

private static int numbits = 5 * 5; //经纬度单独编码长度

private static double minLat;  
private static double minLng;

private final static char[] digits = {&#8216;0&#8217;, &#8216;1&#8217;, &#8216;2&#8217;, &#8216;3&#8217;, &#8216;4&#8217;, &#8216;5&#8217;, &#8216;6&#8217;, &#8216;7&#8217;, &#8216;8&#8217;,  
&#8216;9&#8217;, &#8216;b&#8217;, &#8216;c&#8217;, &#8216;d&#8217;, &#8216;e&#8217;, &#8216;f&#8217;, &#8216;g&#8217;, &#8216;h&#8217;, &#8216;j&#8217;, &#8216;k&#8217;, &#8216;m&#8217;, &#8216;n&#8217;, &#8216;p&#8217;,  
&#8216;q&#8217;, &#8216;r&#8217;, &#8216;s&#8217;, &#8216;t&#8217;, &#8216;u&#8217;, &#8216;v&#8217;, &#8216;w&#8217;, &#8216;x&#8217;, &#8216;y&#8217;, &#8216;z&#8217;};

//定义编码映射关系  
final static HashMap<Character, Integer> lookup = new HashMap<Character, Integer>();

//初始化编码映射内容  
static {  
int i = 0;  
for (char c : digits) {  
lookup.put(c, i++);  
}  
}

public GeoHash() {  
setMinLatLng();  
}

public String encode(double lat, double lon) {  
BitSet latbits = getBits(lat, -90, 90);  
BitSet lonbits = getBits(lon, -180, 180);  
StringBuilder buffer = new StringBuilder();  
for (int i = 0; i < numbits; i++) {  
buffer.append((lonbits.get(i)) ? &#8216;1&#8217; : &#8216;0&#8217;);  
buffer.append((latbits.get(i)) ? &#8216;1&#8217; : &#8216;0&#8217;);  
}  
String code = base32(Long.parseLong(buffer.toString(), 2));  
//Log.i("okunu", "encode lat = " + lat + " lng = " + lon + " code = " + code);  
return code;  
}

public ArrayList<String> getAroundGeoHash(double lat, double lon) {  
//Log.i("okunu", "getArroundGeoHash lat = " + lat + " lng = " + lon);  
ArrayList<String> list = new ArrayList<>();  
double uplat = lat + minLat;  
double downLat = lat &#8211; minLat;

double leftlng = lon &#8211; minLng;  
double rightLng = lon + minLng;

String leftUp = encode(uplat, leftlng);  
list.add(leftUp);

String leftMid = encode(lat, leftlng);  
list.add(leftMid);

String leftDown = encode(downLat, leftlng);  
list.add(leftDown);

String midUp = encode(uplat, lon);  
list.add(midUp);

String midMid = encode(lat, lon);  
list.add(midMid);

String midDown = encode(downLat, lon);  
list.add(midDown);

String rightUp = encode(uplat, rightLng);  
list.add(rightUp);

String rightMid = encode(lat, rightLng);  
list.add(rightMid);

String rightDown = encode(downLat, rightLng);  
list.add(rightDown);

//Log.i("okunu", "getArroundGeoHash list = " + list.toString());  
return list;  
}

//根据经纬度和范围，获取对应的二进制  
private BitSet getBits(double lat, double floor, double ceiling) {  
BitSet buffer = new BitSet(numbits);  
for (int i = 0; i < numbits; i++) {  
double mid = (floor + ceiling) / 2;  
if (lat >= mid) {  
buffer.set(i);  
floor = mid;  
} else {  
ceiling = mid;  
}  
}  
return buffer;  
}

//将经纬度合并后的二进制进行指定的32位编码  
private String base32(long i) {  
char[] buf = new char[65];  
int charPos = 64;  
boolean negative = (i < 0);  
if (!negative) {  
i = -i;  
}  
while (i <= -32) {  
buf[charPos&#8211;] = digits[(int) (-(i % 32))];  
i /= 32;  
}  
buf[charPos] = digits[(int) (-i)];  
if (negative) {  
buf[&#8211;charPos] = &#8216;-&#8216;;  
}  
return new String(buf, charPos, (65 &#8211; charPos));  
}

private void setMinLatLng() {  
minLat = MAXLAT &#8211; MINLAT;  
for (int i = 0; i < numbits; i++) {  
minLat /= 2.0;  
}  
minLng = MAXLNG &#8211; MINLNG;  
for (int i = 0; i < numbits; i++) {  
minLng /= 2.0;  
}  
}

//根据二进制和范围解码  
private double decode(BitSet bs, double floor, double ceiling) {  
double mid = 0;  
for (int i = 0; i < bs.length(); i++) {  
mid = (floor + ceiling) / 2;  
if (bs.get(i)) {  
floor = mid;  
} else {  
ceiling = mid;  
}  
}  
return mid;  
}

//对编码后的字符串解码  
public double[] decode(String geohash) {  
StringBuilder buffer = new StringBuilder();  
for (char c : geohash.toCharArray()) {  
int i = lookup.get(c) + 32;  
buffer.append(Integer.toString(i, 2).substring(1));  
}

BitSet lonset = new BitSet();  
BitSet latset = new BitSet();

//偶数位，经度  
int j = 0;  
for (int i = 0; i < numbits * 2; i += 2) {  
boolean isSet = false;  
if (i < buffer.length()) {  
isSet = buffer.charAt(i) == &#8216;1&#8217;;  
}  
lonset.set(j++, isSet);  
}

//奇数位，纬度  
j = 0;  
for (int i = 1; i < numbits * 2; i += 2) {  
boolean isSet = false;  
if (i < buffer.length()) {  
isSet = buffer.charAt(i) == &#8216;1&#8217;;  
}  
latset.set(j++, isSet);  
}

double lon = decode(lonset, -180, 180);  
double lat = decode(latset, -90, 90);

return new double[]{lat, lon};  
}

public static void main(String[] args) {  
GeoHash geohash = new GeoHash();  
String s = geohash.encode(39.923201, 116.390705);  
System.out.println("geohash：" + s);  
ArrayList<String> aroundGeoHash = geohash.getAroundGeoHash(39.923201, 116.390705);  
for (String s1 : aroundGeoHash) {  
System.out.println("aroundGeoHash：" + s1);  
}  
double[] geo = geohash.decode(s);  
System.out.println(geo[0] + " " + geo[1]);  
}  
}

```

参考：https://www.jianshu.com/p/2fd0cf12e5ba

 [1]: http://www.bridgeli.cn/archives/618