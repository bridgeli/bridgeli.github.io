---
title: MyBatis下最好的分页实现：mybatis-paginator使用入门
author: Bridge Li
type: post
date: 2015-04-26T15:10:02+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
tags:
  - mybatis
  - paginator
  - 分页

---
前两天写一个项目，发现在MyBatis下一个最好的分页实现类库mybatis-paginator，今天就写一篇其入门教程供大家参考。

1. 先引入maven依赖

```

<dependency>  
<groupId>com.github.miemiedev</groupId>  
<artifactId>mybatis-paginator</artifactId>  
<version>1.2.15</version>  
</dependency>

```

从这个依赖中，我们可以看到 1 他是mybatis的一个插件，2 其开源在GitHub上，感兴趣的可以去GitHub上搜源码看看，既然是mybatis的插件，下一步肯定就是和mybatis的集成了.

2. 和mybatis集成

①. 新建一个mybatis.xml的配置文件，其内容如下：

```

<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  
<settings>  
<setting name="cacheEnabled" value="true" />  
<setting name="lazyLoadingEnabled" value="true" />  
</settings>

<plugins>  
<plugin  
interceptor="com.github.miemiedev.mybatis.paginator.OffsetLimitInterceptor">  
<property name="dialectClass"  
value="com.github.miemiedev.mybatis.paginator.dialect.MySQLDialect" />  
<property name="asyncTotalCount" value="true" />  
</plugin>  
</plugins>

</configuration>

```

②. 和spring集成，即在spring-mybatis.xml中引入该文件

```

&#8230;&#8230;

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
<property name="dataSource" ref="dataSource"></property>  
<property name="configLocation" value="classpath:mybatis.xml" />  
<property name="typeAliasesPackage" value="cn.bridgeli.demo.entity" />  
<property name="mapperLocations" value="classpath:cn/bridgeli/demo/mapper/*.xml" />  
</bean>

&#8230;&#8230;

```

因为这个文件太长，就不贴全文了，仅把位置这一块贴出来，当我们把这些配置工作做好之后，下一步就是如何使用了。

3. 分页工具类的封装

```

package com.xmjr.mediastatis.util;

import java.util.HashMap;  
import java.util.List;  
import java.util.Map;

import org.apache.commons.lang.math.NumberUtils;

import com.github.miemiedev.mybatis.paginator.domain.PageBounds;  
import com.github.miemiedev.mybatis.paginator.domain.PageList;  
import com.github.miemiedev.mybatis.paginator.domain.Paginator;

public class PagingUtil {

public static PageBounds getPageBounds(Map<String, String> paramMap) {

String paramPage = paramMap.get("page");  
String paramPageSize = PropertiesUtil.getProperties("pageSize");

int page = NumberUtils.toInt(paramPage, 1); // 页号  
int pageSize = NumberUtils.toInt(paramPageSize, 10); // 每页数据条数

// String sortString = "age.asc,gender.desc";//如果你想排序的话逗号分隔可以排序多列  
PageBounds pageBounds = new PageBounds(page, pageSize);

pageBounds.setAsyncTotalCount(true);  
pageBounds.setContainsTotalCount(true);

return pageBounds;  
}

public static Map<String, Object> toPageInfo(List<Map<String, Object>> detailList) {

PageList<Map<String, Object>> value = (PageList<Map<String, Object>>) detailList;

Map<String, Object> pageInfo = new HashMap<String, Object>();  
Paginator paginator = value.getPaginator();  
pageInfo.put("totalCount", paginator.getTotalCount());  
pageInfo.put("totalPages", paginator.getTotalPages());  
pageInfo.put("page", paginator.getPage());  
pageInfo.put("limit", paginator.getLimit());  
pageInfo.put("items", value);

pageInfo.put("startRow", paginator.getStartRow());  
pageInfo.put("endRow", paginator.getEndRow());

pageInfo.put("offset", paginator.getOffset());

pageInfo.put("slider", paginator.getSlider());

pageInfo.put("prePage", paginator.getPrePage());  
pageInfo.put("nextPage", paginator.getNextPage());

pageInfo.put("firstPage", paginator.isFirstPage());  
pageInfo.put("hasNextPage", paginator.isHasNextPage());  
pageInfo.put("hasPrePage", paginator.isHasPrePage());  
pageInfo.put("lastPage", paginator.isLastPage());

return pageInfo;  
}

}

```

至于封装的这些字段什么意思，大家可以自己去网上搜一些资料看看，其实很多都是见名知意的，当然这个工具类可能不是在大家的项目中可以直接使用的，如果要使用请根据具体情况做修改，修改的主要是toPageInfo方法的参数和第一行而已。好了，当这些准备工作做好之后就可以直接使用了，下面就说一些怎么使用。

4. mybatis-paginator的使用

其使用倒是非常简单，我们只需要在service层中构造我们的分页对象：PageBounds即可，然后把这个对象作为一个参数传到dao层，也就是说dao层除了以前的参数，再新加一个参数pageBounds，其余的mapper.xml等等文件该怎么写就怎么写，和以前可以说一点差别都没有，然后把返回值在封装成一个pageInfo对象，那么pageInfo对象里面就封装了所有我们分页所需要的数据，大家根据具体情况就可以做前端的分页了。例如老夫的实现：

```

List<Map<String, Object>> itemInfos = itemMapper.getItems(param, pageBounds);  
Map<String, Object> pageInfo = PagingUtil.toPageInfo(itemInfos);  
LOG.info("totalCount: " + pageInfo.get("totalCount"));

```

因为这只是一个入门教程，所以更详细的资料大家可以参考：http://blog.csdn.net/szwangdf/article/details/27859847，老夫窃以为这小伙写的不错，当然大家也可以自己去网上搜其他资料，关于这个插件怎么用的教程其实还是蛮多的。