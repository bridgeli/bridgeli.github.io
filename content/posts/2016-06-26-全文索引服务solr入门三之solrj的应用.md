---
title: 全文索引服务solr入门三之solrJ的应用
author: Bridge Li
type: post
date: 2016-06-26T13:52:32+00:00

categories:
  - 全文索引
tags:
  - Lucene
  - solr
  - solrj
  - spring

---
三. 使用solrJ和spring集成

再[上一篇][1]和[上上一篇][2]文章中我们先搭建了一个solr服务器和学习了solr服务器后台的使用，这一次我们将直接进入实战：和spring集成，在继承之前我们先看看所需要的solr的jar文件都是那些（spring的那些大家就自己玩吧，我相信都知道的）

1. 所需的jar文件

直接上图片，就是图上的这些图片，当然大家可以自己找maven依赖（jar文件这个最简单了，没有的话一定会报classnotfoundException，加上就好了）

[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/06/solrJ1-300x217.png" alt="solrJ1" width="300" height="217" class="alignnone size-medium wp-image-285" />][3]  
[<img loading="lazy" decoding="async" src="https://www.bridgeli.cn/wp-content/uploads/2016/06/solrJ2-300x131.png" alt="solrJ2" width="300" height="131" class="alignnone size-medium wp-image-286" />][4]

2. spring的配置

```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"  
xmlns:context="http://www.springframework.org/schema/context"  
xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"  
xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
http://www.springframework.org/schema/mvc  
http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.1.xsd  
http://www.springframework.org/schema/aop  
http://www.springframework.org/schema/aop/spring-aop-3.1.xsd  
http://www.springframework.org/schema/tx  
http://www.springframework.org/schema/tx/spring-tx-3.1.xsd ">  
<!&#8211; 配置扫描包 &#8211;>  
<context:component-scan base-package="cn.bridgeli"/>  
<!&#8211; 配置注解驱动 &#8211;>  
<mvc:annotation-driven/>  
<!&#8211; jsp视图解析器 &#8211;>  
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" >  
<!&#8211; 前缀 &#8211;>  
<property name="prefix" value="/WEB-INF/jsp/"></property>  
<!&#8211; 后缀 &#8211;>  
<property name="suffix" value=".jsp"></property>  
</bean>  
<!&#8211; 单机版solr &#8211;>  
<bean class="org.apache.solr.client.solrj.impl.HttpSolrServer">  
<constructor-arg name="baseURL" value="http://localhost:8080/solr/"></constructor-arg>  
</bean>  
<!&#8211; 集群版SolrCloud &#8211;>  
<!&#8211;  
<bean class="org.apache.solr.client.solrj.impl.CloudSolrServer">  
<constructor-arg name="zkHost" value="127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183"></constructor-arg>  
<property name="defaultCollection" value="collection2"></property>  
</bean>  
&#8211;>  
</beans>

```

简单吧，大家只要注意到单机版就行了，因为我们这次只用到了单机版，下面就要看源码实现了

3. 具体代码实现

①. controller

```

package cn.bridgeli.controller;

import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.web.bind.annotation.RequestMapping;

import cn.bridgeli.model.ResultModel;  
import cn.bridgeli.service.ProductService;  
@Controller  
public class ProductController {

@Autowired  
private ProductService productService;

@RequestMapping("/list")  
public String queryProduct(String queryString, String catalog_name, String price,  
String sort, Integer page, Model model) throws Exception {  
//查询商品列表  
ResultModel resultModel = productService.queryProduct(queryString, catalog_name, price, sort, page);  
//列表传递给jsp  
model.addAttribute("result", resultModel);  
//参数回显  
model.addAttribute("queryString", queryString);  
model.addAttribute("caltalog_name", catalog_name);  
model.addAttribute("price", price);  
model.addAttribute("sort", sort);  
model.addAttribute("page", page);

return "product_list";  
}  
}

```

②. service

```

package cn.bridgeli.service;

import org.apache.solr.client.solrj.SolrQuery;  
import org.apache.solr.client.solrj.SolrQuery.ORDER;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Service;

import cn.bridgeli.common.Commons;  
import cn.bridgeli.dao.ProductDao;  
import cn.bridgeli.model.ResultModel;  
@Service  
public class ProductServiceImpl implements ProductService {

@Autowired  
private ProductDao productDao;

@Override  
public ResultModel queryProduct(String queryString, String caltalog_name,  
String price, String sort, Integer page) throws Exception {  
//拼装查询条件  
SolrQuery query = new SolrQuery();  
//查询条件  
if (null != queryString && !"".equals(queryString)) {  
query.setQuery(queryString);  
} else {  
query.setQuery("\*:\*");  
}  
//商品分类名称过滤  
if (null != caltalog_name && !"".equals(caltalog_name)) {  
query.addFilterQuery("category:" + caltalog_name);  
}  
//价格区间过滤  
if (null != price && !"".equals(price)) {  
String[] strings = price.split("-");  
query.addFilterQuery("price:["+strings[0]+" TO "+strings[1]+"]");  
}  
//排序条件  
if ("1".equals(sort)) {  
query.setSort("price", ORDER.desc);  
} else {  
query.setSort("price", ORDER.asc);  
}  
//分页处理  
if (null == page) {  
page = 1;  
}  
//start  
int start = (page-1) * Commons.PAGE_SIZE;  
query.setStart(start);  
query.setRows(Commons.PAGE_SIZE);

//高亮设置  
query.setHighlight(true);  
query.addHighlightField("name");  
query.setHighlightSimplePre("<span style="color:red">");  
query.setHighlightSimplePost("</span>");

//查询商品列表  
ResultModel resultModel = productDao.queryProduct(query);  
//计算总页数  
long recordCount = resultModel.getRecordCount();  
int pages = (int) (recordCount/Commons.PAGE_SIZE);  
if (recordCount % Commons.PAGE_SIZE > 0) {  
pages ++;  
}  
resultModel.setPageCount(pages);  
resultModel.setCurPage(page);  
return resultModel;  
}  
}

```

这里只列出了实现，结果过于简单就算了，dao也会一样

③. dao

```

package cn.bridgeli.dao;

import java.util.ArrayList;  
import java.util.List;  
import java.util.Map;

import org.apache.solr.client.solrj.SolrQuery;  
import org.apache.solr.client.solrj.SolrServer;  
import org.apache.solr.client.solrj.response.QueryResponse;  
import org.apache.solr.common.SolrDocument;  
import org.apache.solr.common.SolrDocumentList;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Repository;

import cn.bridgeli.model.ProductModel;  
import cn.bridgeli.model.ResultModel;  
@Repository  
public class ProductDaoImpl implements ProductDao {

@Autowired  
private SolrServer solrServer;

@Override  
public ResultModel queryProduct(SolrQuery query) throws Exception {

ResultModel resultModel = new ResultModel();  
//根据query对象查询商品列表  
QueryResponse queryResponse = solrServer.query(query);  
SolrDocumentList solrDocumentList = queryResponse.getResults();  
//取查询结果的总数量  
resultModel.setRecordCount(solrDocumentList.getNumFound());  
List<ProductModel> productList = new ArrayList<>();  
//遍历查询结果  
for (SolrDocument solrDocument : solrDocumentList) {  
//取商品信息  
ProductModel productModel = new ProductModel();  
productModel.setId((String) solrDocument.get("id"));  
//取高亮显示  
String productName = "";  
Map<String, Map<String, List<String>>> highlighting = queryResponse.getHighlighting();  
List<String> list = highlighting.get(solrDocument.get("id")).get("name");  
if (null != list) {  
productName = list.get(0);  
} else {  
productName = (String) solrDocument.get("name");  
}  
productModel.setName(productName);  
productModel.setPrice((float) solrDocument.get("price"));  
productModel.setCategory((String) solrDocument.get("category"));  
productModel.setUrl((String) solrDocument.get("url"));  
//添加到商品列表  
productList.add(productModel);  
}  
//商品列表添加到resultmodel中  
resultModel.setProductList(productList);  
return resultModel;  
}

}

```

④. model

有两个分别是：

```

package cn.bridgeli.model;

public class ProductModel {  
// 商品编号  
private String id;  
// 商品名称  
private String name;  
// 商品分类名称  
private String category;  
// 价格  
private float price;  
// 商品描述  
private String content;  
// 图片名称  
private String url;

//setXX and getXX  
}

package cn.bridgeli.model;

import java.util.List;

public class ResultModel {  
// 商品列表  
private List<ProductModel> productList;  
// 商品总数  
private Long recordCount;  
// 总页数  
private int pageCount;  
// 当前页  
private int curPage;

//setXX and getXX  
}

```

4. 数据库文件

```

CREATE TABLE \`products\` (  
\`pid\` int(11) NOT NULL AUTO_INCREMENT COMMENT &#8216;商品编号&#8217;,  
\`name\` varchar(255) DEFAULT NULL COMMENT &#8216;商品名称&#8217;,  
\`catalog\` int(11) DEFAULT NULL COMMENT &#8216;商品分类ID&#8217;,  
\`catalog_name\` varchar(50) DEFAULT NULL COMMENT &#8216;商品分类名称&#8217;,  
\`price\` double DEFAULT NULL COMMENT &#8216;价格&#8217;,  
\`number\` int(11) DEFAULT NULL COMMENT &#8216;数量&#8217;,  
\`description\` longtext COMMENT &#8216;商品描述&#8217;,  
\`picture\` varchar(255) DEFAULT NULL COMMENT &#8216;图片名称&#8217;,  
\`release_time\` datetime DEFAULT NULL COMMENT &#8216;上架时间&#8217;,  
PRIMARY KEY (\`pid\`)  
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```

好了，只要初始化上数据，我们的一个单机版solr搜索服务器中完成了，开始庆祝吧。

 [1]: https://www.bridgeli.cn/archives/283 "全文索引服务solr入门二之认识管理后台"
 [2]: https://www.bridgeli.cn/archives/277 "全文索引服务solr入门一之单机版服务器搭建"
 [3]: https://www.bridgeli.cn/wp-content/uploads/2016/06/solrJ1.png
 [4]: https://www.bridgeli.cn/wp-content/uploads/2016/06/solrJ2.png