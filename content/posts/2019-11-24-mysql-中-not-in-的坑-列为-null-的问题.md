---
title: MySQL 中 NOT IN 的坑 — 列为 null 的问题
author: Bridge Li
type: post
date: 2019-11-24T03:42:36+00:00

categories:
  - MySQL
tags:
  - not in
---
前一段时间在公司做一个小功能的时候，统计一下某种情况下有多少条数据，然后修改的问题，当时感觉很简单，写了一个如下的 SQL:

```

SELECT COUNT(*) FROM t1 where tl.c1 not IN (SELECT t2.c1 FROM t2);

```

预期的结果是：有多少条数据在 t1 中，同时不在 t2 中，结果为：0，也就是 t1 中数据都在 t2 中，但是很容易就发现某些数据在 t1 中不在 t2 中，所以就感觉很奇怪，这个 SQL 看着也没问题啊。经过一番查询原来是因为 t2 的 c1 字段包含了 null 值，修改如下两种形式都可以得到预期的结果：

```

SELECT COUNT(*) FROM t1 LEFT JOIN t2 ON t1.c1 = t2.c1 WHERE t2.c1 IS NULL OR t2.c1 = &#8221;;

```

或者

```

select COUNT(*) from t1 where t1.c1 not in (  
select t2.c1 from t2 where t2.c1 is not null AND t2.c1 != &#8221;  
);

```

所以都是 null 引起的（为了避免错误我把空串也加上了），原因是 not in 的实现原理是，对每一个 t1.c1 和每一个 t2.c1 （括号内的查询结果）进行不相等比较（!=）。

```

foreach c1 in t2:  
if t1.c1 != c1:  
continue  
else:  
return false  
return true

```

而 SQL 中任意 !=null 的运算结果都是 false，所以如果 t2 中存在一个 null，not in 的查询永远都会返回 false，即查询结果为空。