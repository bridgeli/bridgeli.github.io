---
title: 介绍一个 Mybatis 插件：mybatis-generator-plugin
author: Bridge Li
type: post
date: 2018-04-29T09:37:55+00:00

categories:
  - Java
tags:
  - generator
  - mybatis
  - mybatis-generator-plugin
  - MySQL
  - plugin

---
在实际开发中，我们都是先建表，然后根据表生成对应的 Java 类，现在很流行的 ORMapping 框架是: Mybatis，所以我们需要生成 entity、mapper 和 xml，我们都知道有一个插件是：mybatis-generator，使用它就可以很方便的生成这些结构化的重复性基础性的代码，但是他有一个问题，生成的查询没有分页，所以很烦。然后我搜索了一些资料，重新封装了一下，重新命名为：mybatis-generator-plugin，具体源码放在了 GitHub 上：

```

https://github.com/bridgeli/mybatis-generator-plugin

```

大家可以 clone 下来，deploy 到自己公司的私服，或者 install 到本地使用。具体就是在 pom 文件中。具体用法

1. 在 pom 文件中添加如下内容：

```

<properties>  
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
</properties>

<build>  
<plugins>  
<plugin>  
<groupId>org.mybatis.generator</groupId>  
<artifactId>mybatis-generator-maven-plugin</artifactId>  
<version>1.3.5</version>  
<dependencies>  
<dependency>  
<groupId>cn.bridgeli</groupId>  
<artifactId>mybatis-generator-plugin</artifactId>  
<version>0.0.1-SNAPSHOT</version>  
</dependency>  
</dependencies>  
<configuration>  
<verbose>true</verbose>  
<overwrite>true</overwrite>  
<configurationFile>src/main/resources/generatorConfig.xml</configurationFile>  
</configuration>  
</plugin>  
</plugins>  
</build>

```

其中：configurationFile 配置 generatorConfig.xml 文件的位置，我们生成的对应的文件，都是根据这个文件来的。所以第二步就是：

2. 在对应的文件下添加该文件，它的模板如下：

```

<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE generatorConfiguration  
PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"  
"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

<!&#8211; 请替换本地jar路径 此文件不要上传 &#8211;>  
<classPathEntry  
location="/Users/bridgeli/.m2/repository/mysql/mysql-connector-java/5.1.33/mysql-connector-java-5.1.33.jar" />

<!&#8211; mvn mybatis-generator:generate &#8211;>  
<context id="oneHour"  
targetRuntime="cn.bridgeli.plugin.IntrospectedTableMyBatis3ImplExt">  
<property name="suppressAllComments" value="true" />  
<property name="useActualColumnNames" value="false" />  
<!&#8211; 格式化java代码 &#8211;>  
<property name="javaFormatter"  
value="org.mybatis.generator.api.dom.DefaultJavaFormatter" />  
<!&#8211; 格式化XML代码 &#8211;>  
<property name="xmlFormatter"  
value="org.mybatis.generator.api.dom.DefaultXmlFormatter" />

<!&#8211; 配置插件 &#8211;>  
<plugin type="cn.bridgeli.mybatis.MySQLLimitPlugin"></plugin>

<commentGenerator>  
<!&#8211; 是否去除自动生成的注释 true：是 ： false:否 &#8211;>  
<property name="suppressAllComments" value="true" />  
</commentGenerator>  
<!&#8211;数据库连接的信息：驱动类、连接地址、用户名、密码 &#8211;>  
<jdbcConnection driverClass="com.mysql.jdbc.Driver"  
connectionURL="jdbc:mysql://ip:port/databasename" userId="username"  
password="password">  
</jdbcConnection>

<javaTypeResolver>  
<property name="forceBigDecimals" value="false" />  
</javaTypeResolver>

<!&#8211; 配置model生成位置 &#8211;>  
<javaModelGenerator targetPackage="cn.bridgeli.entity"  
targetProject="src/main/java">  
<property name="enableSubPackages" value="true" />  
<property name="trimStrings" value="true" />  
</javaModelGenerator>

<!&#8211; 配置sqlmap生成位置 &#8211;>  
<sqlMapGenerator targetPackage="cn/bridgeli/mapper"  
targetProject="src/main/resources">  
<property name="enableSubPackages" value="true" />  
</sqlMapGenerator>

<!&#8211; 配置mapper接口生成位置 &#8211;>  
<javaClientGenerator type="XMLMAPPER"  
targetPackage="cn.bridgeli.mapper" targetProject="src/main/java">  
<property name="enableSubPackages" value="true" />  
</javaClientGenerator>

<!&#8211; 配置表信息 &#8211;>  
<table tableName="table_name" enableCountByExample="true"  
domainObjectName="DomainObjectName" enableDeleteByExample="false"  
enableSelectByExample="true" enableUpdateByExample="true">  
<generatedKey column="ID" sqlStatement="MySQL"  
identity="true" />  
</table>  
</context>

</generatorConfiguration>

```

这里面需要说明就是：

```

<plugin type="cn.bridgeli.mybatis.MySQLLimitPlugin"></plugin>

```

这个插件，他就会在生成的文件中添加添加分页，所以第三步：

3. 修改相应的配置（具体配置的含义可以参考，参考链接 1 和 2）。

4. 在 pom 所在的路径下，执行如下命令就可生成想要的代码：

```

mvn mybatis-generator:generate

```

5. 具体的使用方法，这就很简单了，方法一：

```

XxxExample example = new XxxExample();  
&#8230;  
example.setLimit(10); // page size limit  
example.setOffset(20); // offset  
List<Xxx> list = xxxMapper.selectByExample(example);

```

以上代码运行时执行的 SQL 是：

```

select &#8230; limit 20, 10

```

方法二：

```

XxxExample example = new XxxExample();  
&#8230;  
example.setLimit(10); // limit  
List<Xxx> list = xxxMapper.selectByExample(example);

```

以上代码运行时执行的 SQL 是：

```

select &#8230; limit 10

```

最后欢迎大家 follow 我的 GitHub，或者 star 其中的项目

https://github.com/bridgeli

参考链接：  
1. http://www.mybatis.org/generator/  
2. http://www.cnblogs.com/coderland/p/5902882.html  
3. https://blog.csdn.net/xiao__gui/article/details/51333693