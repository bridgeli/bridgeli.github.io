---
title: Spring aop应用之实现数据库读写分离
author: Bridge Li
type: post
date: 2016-12-31T12:12:51+00:00

categories:
  - Java
tags:
  - AOP
  - spring
  - 数据库
  - 读写分离

---
去年五月份的时候曾经写过一篇：[Spring加Mybatis实现MySQL数据库主从读写分离][1]，实现的原理是配置了多套数据源，相应的sqlsessionfactory，transactionmanager和事务代理各配置了一套，如果从库或数据库有多个的时候，需要配置的信息会越来越多，远远不够优雅，在我们编程界有一个规范：约定优于配置。所以就用Sping的aop实现了一个简单的数据库分离方案，具体实现代码放在了Github上，地址如下：

```

https://github.com/bridgeli/practical-util/tree/master/src/main/java/cn/bridgeli/datasource

```

读者如果想使用再简单的方法就是把这个代码download下来，放到自己的项目里面，当然更优雅的方式是：打成jar包，放到项目里面了，具体打jar的方法，老夫就不在这里多说了，相信看这篇文章的读者肯定都会了。当然仅仅有这份代码，他们是不会自动生效的，既然是使用Spring的Aop实现数据库读写分离，所以肯定会有牵涉到Aop的配置了，所以在spring-mybatis.xml中有如下配置：

```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"  
xmlns:p="http://www.springframework.org/schema/p" xmlns:tx="http://www.springframework.org/schema/tx"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xsi:schemaLocation="  
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd  
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd  
http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">

<!&#8211; 配置写数据源 &#8211;>  
<bean id="masterDataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">  
<property name="driverClassName" value="${bridgeli.jdbc.driver}" />  
<property name="url" value="${bridgeli.jdbc.url}" />  
<property name="username" value="${bridgeli.jdbc.username}" />  
<property name="password" value="${bridgeli.jdbc.password}" />  
<property name="validationQuery" value="${bridgeli.jdbc.validationQuery}" />  
<property name="initialSize" value="1" />  
<property name="maxActive" value="20" />  
<property name="minIdle" value="0" />  
<property name="maxWait" value="60000" />  
<property name="testOnBorrow" value="false" />  
<property name="testOnReturn" value="false" />  
<property name="testWhileIdle" value="true" />  
<property name="timeBetweenEvictionRunsMillis" value="60000" />  
<property name="minEvictableIdleTimeMillis" value="25200000" />  
<property name="removeAbandoned" value="true" />  
<property name="removeAbandonedTimeout" value="1800" />  
<property name="logAbandoned" value="true" />  
<property name="filters" value="stat" />  
</bean>

<!&#8211; 配置读数据源 &#8211;>  
<bean id="parentSlaveDataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">  
<property name="driverClassName" value="${bridgeli.jdbc.driver}" />  
<property name="validationQuery" value="${bridgeli.jdbc.validationQuery}" />  
<property name="initialSize" value="1" />  
<property name="maxActive" value="20" />  
<property name="minIdle" value="0" />  
<property name="maxWait" value="60000" />  
<property name="testOnBorrow" value="false" />  
<property name="testOnReturn" value="false" />  
<property name="testWhileIdle" value="true" />  
<property name="timeBetweenEvictionRunsMillis" value="60000" />  
<property name="minEvictableIdleTimeMillis" value="25200000" />  
<property name="removeAbandoned" value="true" />  
<property name="removeAbandonedTimeout" value="1800" />  
<property name="logAbandoned" value="true" />  
<property name="filters" value="stat" />  
</bean>  
<bean id="slaveDataSource1" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close" parent="parentSlaveDataSource">  
<property name="url" value="${bridgeli_slave1.jdbc.url}" />  
<property name="username" value="${bridgeli_slave1.jdbc.username}" />  
<property name="password" value="${bridgeli_slave1.jdbc.password}" />  
</bean>

<bean id="dataSource" class="cn.bridgeli.datasource.MasterSlaveDataSource">  
<property name="targetDataSources">  
<map>  
<entry key-ref="masterDataSource" value-ref="masterDataSource"/>  
<entry key-ref="slaveDataSource1" value-ref="slaveDataSource1"/>  
</map>  
</property>  
<property name="defaultTargetDataSource" ref="masterDataSource"/>  
<property name="masterSlaveSelector" ref="dataSelector"/>  
</bean>

<bean id="dataSelector" class="cn.bridgeli.datasource.MasterSlaveSelectorByPoll">  
<property name="masters">  
<list>  
<ref bean="masterDataSource"/>  
</list>  
</property>  
<property name="slaves">  
<list>  
<ref bean="slaveDataSource1"/>  
</list>  
</property>  
<property name="defaultDataSource" ref="masterDataSource"/>  
</bean>

<aop:aspectj-autoproxy/>

<!&#8211; mybaits 数据工厂 &#8211;>  
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
<property name="dataSource" ref="dataSource" />  
</bean>

<!&#8211; 自动扫描所有注解的路径 &#8211;>  
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
<property name="basePackage" value="cn.bridgeli.mapper" />  
<!&#8211; <property name="sqlSessionFactory" ref="sqlSessionFactory" /> &#8211;>  
<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>  
</bean>

<!&#8211; 数据库切面 &#8211;>  
<bean id="masterSlaveAspect" class="cn.bridgeli.datasource.MasterSlaveAspect">  
<property name="prefixMasters">  
<list>  
<value>save</value>  
<value>update</value>  
<value>delete</value>  
</list>  
</property>  
</bean>  
<aop:config>  
<aop:aspect id="c" ref="masterSlaveAspect">  
<aop:pointcut id="tx" expression="execution(\* cn.bridgeli.service..\*.*(..))"/>  
<aop:before pointcut-ref="tx" method="before"/>  
</aop:aspect>  
</aop:config>

<context:annotation-config />  
<context:component-scan base-package="cn.bridgeli" />  
</beans>

```

这样我们就很优雅的利用Spring的Aop实现了对数据库的读写分离，读的时候走slaveDataSource1这个数据源，写的时候走masterDataSource这个数据源。哎，等等，这里哪里体现了约定优于配置这一规范，他们怎么知道哪些方法走读库哪些走写库？同学你别急，仔细读读这个配置文件，你就会发现在第98行，配置了一个MasterSlaveAspect，也就是说代码里面service层（为什么是service层？）的方法以这里面配置的这些关键字打头，都将会走写库，所以当我们想让一个方法走主库的时候，必须在这个地方添加该方法的前缀或者用这里面已有的前缀，这就要求我们必须约定好走主库的方法的打头，即约定优于配置。

 [1]: https://www.bridgeli.cn/archives/166 "Spring加Mybatis实现MySQL数据库主从读写分离"