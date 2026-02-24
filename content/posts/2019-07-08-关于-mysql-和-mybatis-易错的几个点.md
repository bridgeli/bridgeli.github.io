---
title: 关于 MySQL 和 MyBatis 易错的几个点
author: Bridge Li
type: post
date: 2019-07-08T03:30:12+00:00

categories:
  - Java
tags:
  - mybatis
  - MySQL

---
由于某些不可抗拒力原因，自从开博以来断更了一个月，昨天晚上突然发现竟然解封了，今天立即写一篇小文章感谢党感谢政府感谢人民。话说，这一周有一个实习的同学，在写一个小东西的时候，发现一个问题，排序没有生效，刚好之前我也看过另外一个问题，现在算是总结一下。

1. ORDER BY 不生效

代码大概就是：

```

SELECT * FROM t ORDER BY "id DESC";

```

我们其实可以很明显的看出来这个 SQL 有问题，ORDER BY 的后面多了引号，关键是这个 SQL 报错吗？如果 id 是随便写的一个不存在的列报错吗？答案是都不报错，排序也不生效，大家可以测试一些，这有时候就比较坑了，具体为什么会出现这个错误，后面再说，先说第二个。

2. 分页问题

代码如下：

```

/**  
* 开始页码  
*/  
private String pageSize;  
/**  
* 分页量  
*/  
private String offset;

<if test="pageSize != null">  
<if test="offset != null">  
limit #{offset}, #{pageSize}  
</if>  
<if test="offset == null">  
limit #{pageSize}  
</if>  
</if>

```

这个错误就比较明显了，一般人也不会犯，为什么要单独说一下呢？因为某同事说，分页用 # 会出错，但是我记得我一直用 # 没问题啊，刚好在遇到上面这个问题的时候，顺便测试了一些，还真有错，奇怪啊，其实原因很简单，就是分页的对象的属性有误，我们知道分页的属性，一般都是用 Integer，这个地方却写了 String，真是没事找事啊，一般谁会这么写？个人认为除非脑子有病。下面看第三个问题，也是一个比较诡异的问题。

3. UPDATE

```

UPDATE t SET username = "BridgeLi" AND passwd = "BridgeLi" WHERE id = 1;

```

这个 SQL 有问题吗？会报错吗？这个问题有时候真不太看得出来，其实也很明显，SET 后面的各个列应该是 “,” 相连，而这个用的是 AND，有时候脑子一抽，还真有可能写错。但是会报错吗？这个问题还真不好答复，因为这个 SQL 这么写，相当于：

```

UPDATE t SET username = ("BridgeLi" AND passwd = "BridgeLi") WHERE id = 1;

```

所以就是这个 SQL 有错吗？看网上说的这个是不报错的，但是经过我的测试，和 username 类型相关，username 是 varchar 类型就会报错，是 int 类型就不会(我没有很严格的测试，总之有不报错的情况，就够了，因为那个时候脑子抽了，不报错，大概率的自己一时半会看不出来)，至于这个 username 被更新成了什么，我想大家也可以猜出来，取决于 id 为 1 的行，passwd 这一列的值，然后相与计算。

说完了，这三个报错，说一下为什么会出现这些问题，其实 3 很简单，就是脑子一抽写错了，这个没啥说的；1 和 2 也其实是一个问题，就是对 mybatis 不熟啊，简单说一下。

1. 对于 #{variable} 的变量，mybatis 会将其视为字符串值，在变量替换成功后，缺省地给变量值加上引号：&#8221;variable&#8221;；  
2. 对于 ${variable} 的变量，mybatis 会将其视作直接变量，即在变量替换成功后，不会再给其加上引号：variable

所以在 order by ${name} 中，传入的直接是 name ，不带双引号，因为 order by 不是 = 赋值，如果直接 order by #{name}，结果是 order by &#8220;name&#8221;，自然就不行了

总结，#{variable} 传入字符串，可以在日志查看到传入的参数，需要赋值后使用，可以有效防止 SQL 注入，${variable} 是直接传入变量，在日志查看不到传入的变量，直接在 SQL 中执行，无法防止 SQL 注入，所以，尽量用 #{variable} 格式，但如果不是类似 = 赋值后再使用的 SQL，需要使用 ${variable}。另外网上还说有内需要注意 # 与 $ 的区别，没有太在意过，一般都是和 大于小于 混用的时候，偶尔会写他，不写也不会报错，大家可以自己测试一下。

最后说一下，有同学提到 $ 有 SQL 注入问题啊，你上面也说，我个人的理解就是 SQL 注入都是使你的 WHERE 子句为真，ORDER BY 有什么影响吗？没有啊，所以这个问题并不需要考虑。