---
title: MySQL sort 分页重复数据（转载）
author: Bridge Li
type: post
date: 2019-01-26T14:59:01+00:00

categories:
  - MySQL
tags:
  - 分页
  - 重复数据

---
前两天在写一个东西的时候，测试的同学说发现一个问题，排序分页，第二页和第一页有重复数据，当时我看了一下，确实有这个问题，然后就想到几年前我就曾经遇到过这个问题，淘宝数据库内核月报上也做了说明，所以这个时候就体现出了老程序员的价值：踩过的坑多，坑坑相连也就都成了平地，考虑到很多人不知道这个问题，所以把原文转载过来，以期能够让更多的人看到，原文如下：

1. 背景

6.5 号，小编在 Aliyun 的论坛中发现一位开发者提的一个问题，说 RDS 发现了一个超级大 BUG，吓的小编一身冷汗 = =!! 赶紧来看看，背景是一个 RDS 用户创建了一张表，在一个都是 NULL 值的非索引字段上进行了排序并分页，用户发现第二页和第一页的数据有重复，然后以为是 NULL 值的问题，把这个字段都更新成相同的值，发现问题照旧。详细的信息可以登录阿里云的[官方论坛查看][1]。  
小编进行了尝试，确实如此，并且 5.5 的版本和 5.6 的版本行为不一致，所以，必须要查明原因。

2. 原因调查

在 MySQL 5.6 的版本上，优化器在遇到 order by limit 语句的时候，做了一个优化，即使用了 priority queue。参考伪代码：

```

while (get_next_sortkey())  
{  
if (using priority queue)  
push sort key into queue  
else  
{  
if (no free space in sort_keys buffers)  
{  
sort sort_keys buffer;  
dump sorted sequence to &#8216;tempfile&#8217;;  
dump BUFFPEK describing sequence location into &#8216;buffpek_pointers&#8217;;  
}  
put sort key into &#8216;sort_keys&#8217;;  
}  
}  
if (sort_keys has some elements && dumped at least once)  
sort-dump-dump as above;  
else  
don&#8217;t sort, leave sort_keys array to be sorted by caller

```

使用 priority queue 的目的，就是在不能使用索引有序性的时候，如果要排序，并且使用了limit n，那么只需要在排序的过程中，保留 n 条记录即可，这样虽然不能解决所有记录都需要排序的开销，但是只需要 sort buffer 少量的内存就可以完成排序。  
之所以 5.6 出现了第二页数据重复的问题，是因为 priority queue 使用了堆排序的排序方法，而堆排序是一个不稳定的排序方法，也就是相同的值可能排序出来的结果和读出来的数据顺序不一致。  
5.5 没有这个优化，所以也就不会出现这个问题。

3. 解决方法

①. 索引排序字段  
之前的月报中，我们讨论过三星索引的设计，其中第二条就是利用索引的有序性，如果用户在字段添加上索引，就直接按照索引的有序性进行读取并分页，从而可以规避遇到的这个问题。

②. 正确理解分页  
还是要正确理解分页，分页是建立在排序的基础上，进行了数量范围分割。排序是数据库提供的功能，而分页却是衍生的出来的应用需求。在 MySQL 和 Oracle 的官方文档中提供了 limit n 和 rownum < n 的方法，但却没有明确的定义分页这个概念。还有重要的一点，虽然上面的解决方法可以缓解用户的这个问题，但按照用户的理解，依然还有问题：比如，这个表插入比较频繁，用户查询的时候，在 read-committed 的隔离级别下，第一页和第二页仍然会有重合。

分页一直都有这个问题，我们看分页常用的场景：1)早期的论坛 2)个人交易记录。这些场景都对数据分页都没有非常高的准确性要求。

4. 究竟是不是 BUG

究竟归于 bug 问题还是用户使用理解上的问题？

小编觉得应该分开看待这个问题，如果是排序的问题，那就算是 BUG，如果是分页的这个问题，那它确实完成了 order by 的功能，也完成了 limit n 功能，那就不能说它是 BUG，分页就纯粹变成了用户理解的问题了。

5. 用户在使用数据库的时候常见的一些问题：

①. 不加 order by 的时候的排序问题  
用户在使用 Oracle 或 MySQL 的时候，发现 MySQL 总是有序的，Oracle 却很混乱，这个主要是因为 Oracle 是堆表，MySQL 是索引聚簇表的原因。所以没有 order by 的时候，数据库并不保证记录返回的顺序性，并且不保证每次返回都一致的。

②. 分页问题  
分页重复的问题，就如前面所描述的，分页是在数据库提供的排序功能的基础上，衍生出来的应用需求，数据库并不保证分页的重复问题。

③. NULL 值和空串问题  
不同的数据库对于 NULL 值和空串的理解和处理是不一样的，比如 Oracle NULL 和 NULL 值是无法比较的，既不是相等也不是不相等，是未知的。而对于空串，在插入的时候，MySQL 是一个字符串长度为 0 的空串，而 Oracle 则直接进行NULL值处理。

原文链接：http://mysql.taobao.org/monthly/2015/06/04/

 [1]: https://bbs.aliyun.com/read/248026.html