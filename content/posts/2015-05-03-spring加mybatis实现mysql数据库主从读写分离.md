---
title: Spring加Mybatis实现MySQL数据库主从读写分离
author: Bridge Li
type: post
date: 2015-05-03T15:19:38+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
tags:
  - mybatis
  - MySQL，spring
  - 主从分离
  - 读写分离

---
上周在一个同事的指点下，实现了Spring加Mybatis实现了MySQL的主从读写分离，今天记一下笔记，以供自己今后参考，下面是配置文件的写法。

1. 数据源也就是jdbc.properties，因为是主从读写分离，那么肯定有两个数据源了

```

jdbc.driver=org.mariadb.jdbc.Driver

\# 从库，只读  
slave.jdbc.url=jdbc:mariadb://xxx.xxx.xxx.xxx:3306/xxx?characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&noAccessToProcedureBodies=true&autoReconnect=true  
slave.jdbc.username=xxx  
slave.jdbc.password=xxx

\# 主库，需要写  
master.jdbc.url=jdbc:mariadb://xxx.xxx.xxx.xxx:3306/xxx?characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&noAccessToProcedureBodies=true&autoReconnect=true  
master.jdbc.username=xxx  
master.jdbc.password=xxx

```

这个非常简单和普通的区别不是很大，另外数据库的驱动是：mariadb，动下面就是第二个配置文件spring.xml

2. spring.xml

```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"  
xsi:schemaLocation="  
http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.0.xsd  
">

<!&#8211; Import properties file &#8211;>  
<context:property-placeholder location="classpath:jdbc.properties" />

<!&#8211; Auto Scan &#8211;>  
<context:component-scan base-package="cn.bridgeli.demo" />  
</beans>

```

这个文件很简单，有两个作用，①. 引入数据源配置文件；②. spring扫描的文件的路径

3. spring-mybatis.xml配置文件

```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation="  
http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/tx  
http://www.springframework.org/schema/tx/spring-tx-3.0.xsd  
http://www.springframework.org/schema/aop  
http://www.springframework.org/schema/aop/spring-aop-3.0.xsd  
">

<bean id="slaveDataSourceImpl" class="com.jolbox.bonecp.BoneCPDataSource" destroy-method="close">  
<property name="driverClass" value="${jdbc.driver}" />  
<property name="jdbcUrl" value="${slave.jdbc.url}" />  
<property name="username" value="${slave.jdbc.username}" />  
<property name="password" value="${slave.jdbc.password}" />

<!&#8211; 检查数据库连接池中空闲连接的间隔时间，单位是分，默认值：240，如果要取消则设置为0 &#8211;>  
<property name="idleConnectionTestPeriodInMinutes" value="10" />  
<!&#8211; 连接池中未使用的链接最大存活时间，单位是分，默认值：60，如果要永远存活设置为0 &#8211;>  
<property name="idleMaxAgeInMinutes" value="10" />  
<!&#8211; 每个分区最大的连接数 &#8211;>  
<property name="maxConnectionsPerPartition" value="20" />  
<!&#8211; 每个分区最小的连接数 &#8211;>  
<property name="minConnectionsPerPartition" value="10" />  
<!&#8211; 分区数 ，默认值2，最小1，推荐3-4，视应用而定 &#8211;>  
<property name="partitionCount" value="3" />  
<!&#8211; 每次去拿数据库连接的时候一次性要拿几个,默认值：2 &#8211;>  
<property name="acquireIncrement" value="3" />  
<!&#8211; 缓存prepared statements的大小，默认值：0 &#8211;>  
<property name="statementsCacheSize" value="50" />  
<!&#8211; 在做keep-alive的时候的SQL语句 &#8211;>  
<property name="connectionTestStatement" value="select 1 from dual" />  
<!&#8211; 在每次到数据库取连接的时候执行的SQL语句，只执行一次 &#8211;>  
<property name="initSQL" value="select 1 from dual" />  
<property name="closeConnectionWatch" value="false" />  
<property name="logStatementsEnabled" value="true" />  
<property name="transactionRecoveryEnabled" value="true" />  
</bean>

<bean id="masterDataSourceImpl" class="com.jolbox.bonecp.BoneCPDataSource" destroy-method="close">  
<property name="driverClass" value="${jdbc.driver}" />  
<property name="jdbcUrl" value="${master.jdbc.url}" />  
<property name="username" value="${master.jdbc.username}" />  
<property name="password" value="${master.jdbc.password}" />

<!&#8211; 检查数据库连接池中空闲连接的间隔时间，单位是分，默认值：240，如果要取消则设置为0 &#8211;>  
<property name="idleConnectionTestPeriodInMinutes" value="10" />  
<!&#8211; 连接池中未使用的链接最大存活时间，单位是分，默认值：60，如果要永远存活设置为0 &#8211;>  
<property name="idleMaxAgeInMinutes" value="10" />  
<!&#8211; 每个分区最大的连接数 &#8211;>  
<property name="maxConnectionsPerPartition" value="20" />  
<!&#8211; 每个分区最小的连接数 &#8211;>  
<property name="minConnectionsPerPartition" value="10" />  
<!&#8211; 分区数 ，默认值2，最小1，推荐3-4，视应用而定 &#8211;>  
<property name="partitionCount" value="3" />  
<!&#8211; 每次去拿数据库连接的时候一次性要拿几个,默认值：2 &#8211;>  
<property name="acquireIncrement" value="3" />  
<!&#8211; 缓存prepared statements的大小，默认值：0 &#8211;>  
<property name="statementsCacheSize" value="50" />  
<!&#8211; 在做keep-alive的时候的SQL语句 &#8211;>  
<property name="connectionTestStatement" value="select 1 from dual" />  
<!&#8211; 在每次到数据库取连接的时候执行的SQL语句，只执行一次 &#8211;>  
<property name="initSQL" value="select 1 from dual" />  
<property name="closeConnectionWatch" value="false" />  
<property name="logStatementsEnabled" value="true" />  
<property name="transactionRecoveryEnabled" value="true" />  
</bean>

<!&#8211; DataSource/Master &#8211;>  
<bean id="masterDataSource"  
class="org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy">  
<property name="targetDataSource" ref="masterDataSourceImpl" />  
</bean>  
<bean id="masterTransactionManager"  
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
<property name="dataSource" ref="masterDataSource" />  
</bean>  
<bean id="masterTransactionTemplate"  
class="org.springframework.transaction.support.TransactionTemplate">  
<property name="transactionManager" ref="masterTransactionManager" />  
</bean>

<!&#8211; DataSource/Slave &#8211;>  
<bean id="slaveDataSource"  
class="org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy">  
<property name="targetDataSource" ref="slaveDataSourceImpl" />  
</bean>  
<bean id="slaveTransactionManager"  
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
<property name="dataSource" ref="slaveDataSource" />  
</bean>  
<bean id="slaveTransactionTemplate"  
class="org.springframework.transaction.support.TransactionTemplate">  
<property name="transactionManager" ref="slaveTransactionManager" />  
</bean>

<!&#8211; Mybatis/Master &#8211;>  
<bean id="masterSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
<property name="dataSource" ref="masterDataSource"></property>  
<property name="configLocation" value="classpath:mybatis.xml" />  
<property name="typeAliasesPackage" value="cn.bridgeli.demo.entity" />  
<property name="mapperLocations">  
<list>  
<value>classpath:cn/bridgeli/demo/mapper/master/*.xml</value>  
</list>  
</property>  
</bean>

<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
<property name="basePackage" value="cn.bridgeli.demo.mapper.master" />  
<property name="sqlSessionFactoryBeanName" value="masterSqlSessionFactory" />  
</bean>

<!&#8211; Mybatis/Slave &#8211;>  
<bean id="slaveSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
<property name="dataSource" ref="slaveDataSource"></property>  
<property name="configLocation" value="classpath:mybatis.xml" />  
<property name="typeAliasesPackage" value="cn.bridgeli.demo.entity" />  
<property name="mapperLocations">  
<list>  
<value>classpath:cn/bridgeli/demo/mapper/slave/*.xml</value>  
</list>  
</property>  
</bean>

<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
<property name="basePackage" value="cn.bridgeli.demo.mapper.slave" />  
<property name="sqlSessionFactoryBeanName" value="slaveSqlSessionFactory" />  
</bean>

<!&#8211; Configuration transaction advice &#8211;>  
<tx:advice id="txAdvice" transaction-manager="masterTransactionManager">  
<tx:attributes>  
<tx:method name="add*" propagation="REQUIRED" />  
<tx:method name="update*" propagation="REQUIRED" />  
<tx:method name="delete*" propagation="REQUIRED" />  
<tx:method name="get*" read-only="true" propagation="SUPPORTS" />  
<tx:method name="list*" read-only="true" propagation="SUPPORTS" />  
</tx:attributes>  
</tx:advice>  
<!&#8211; Configuration transaction aspect &#8211;>  
<aop:config>  
<aop:pointcut id="systemServicePointcut"  
expression="execution(\* cn.bridgeli.demo.service.\*.*(..))" />  
<aop:advisor advice-ref="txAdvice" pointcut-ref="systemServicePointcut" />  
</aop:config>

</beans>

```

这个配置文件老夫是完整的copy了下来，看起来也比较易懂，就不做解释了，需要说明的mybatis下那些dao的接口，分别对应cn.bridgeli.demo.mapper.master、cn.bridgeli.demo.mapper.slave，cn.bridgeli.demo.mapper.master下的这些dao接口是要写的，另一个是读的，这些接口对应的配置文件肯定就是他们对应的文件夹下面的xml文件了，在将来的项目中几乎可以照抄

4. mybatis的配置文件mybatis.xml

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

这个配置文件就一个分页，在前一篇文章中写过，就不多做解释了

5. 最后因为用的数据源的问题，就把pom.xml文件需要引入的几个依赖，也摘出如下

```

<dependency>  
<groupId>com.jolbox</groupId>  
<artifactId>bonecp</artifactId>  
<version>0.8.0.RELEASE</version>  
</dependency>

<dependency>  
<groupId>com.jolbox</groupId>  
<artifactId>bonecp-spring</artifactId>  
<version>0.8.0.RELEASE</version>  
</dependency>

<dependency>  
<groupId>org.mariadb.jdbc</groupId>  
<artifactId>mariadb-java-client</artifactId>  
<version>1.1.7</version>  
</dependency>

```

需要说明的我们用常见的c3p0或者dbcp的数据源也是可以的，至于本例中为什么用了这个没有听说过的jolbox（老夫是没听说过），老夫也不知道，知道他和常见的c3p0和dbcp有什么区别的请留言交流，谢谢