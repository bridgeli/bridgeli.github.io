---
title: MySQL 系统参数 sql_safe_updates 小结
author: Bridge Li
type: post
date: 2019-10-27T05:08:21+00:00

categories:
  - MySQL
tags:
  - sql_safe_updates
---
前一段时间，公司某个项目组某个项目因为失误，出现了一个严重 bug，动态 SQL 导致没有 where 条件，就把数据库某张表里面的数据全部更新了，虽然事后 DBA 同学很给力的恢复了，但是运维的同学讨论，让所有的项目都不许写动态 SQL，必须根据 ID 更新，并写了一个 sonar 插件，扫描代码，发现有动态 SQL 就报 bug，不过还好暂时没有强制要求改，个人认为如果强制要求，这不就是典型的因噎废食吗？没有动态 SQL，这代码量得增加多少？我们研发要不要按照代码量算钱？前一段时间，看《即刻时间》丁奇（原名林晓斌）的 《MySQL 实战 45 讲》里面有一句话关于 sql_safe_updates 的知识点一笔带过，小小研究一下，刚好可以解决这个问题，这是一个小结。

作用：防止忘记添加 WHERE 条件，导致数据被误更新或误删的情况和另外为了提高 SQL 性能，避免更新或删除的时候 WHERE 条件不走索引的情况；不过默认值是：关闭（值为0），所以可能更多的防止忘记添加 WHERE 条件吧，另外这个参数分为会话级别和全局级别。

1. 查看 sql_safe_updates 的值和修改

```

show variables like &#8216;sql_safe_updates&#8217;; &#8212; 会话  
show global variables like &#8216;sql_safe_updates&#8217;; &#8212; 全局

set sql_safe_updates=1; &#8212; 会话  
set global sql_safe_updates=1; &#8212; 全局

```

2. 具体的测试比较简单，就不一张张的贴图了，直接写结论了，大家可以自己测试一下就好了

```

操作 Delete Update

NO WHERE No No

NO WHERE + LIMIT No Yes

WHERE KEY Yes Yes

WHERE KEY + LIMIT Yes Yes

WHERE NOKEY No No

WHERE NOKEY+ LIMIT Yes Yes

WHERE CONSTANT No No

WHERE CONSTANT + LIMIT No Yes

```

其中 KEY 不仅是主键，同样也包括索引字段，CONSTANT 表示 where 1= 1（有些程序员的知识没有更新喜欢这么写，其实可以不用的），Yes 是可以，No 是不可以。

参考资料：https://www.cnblogs.com/kerrycode/p/10569457.html