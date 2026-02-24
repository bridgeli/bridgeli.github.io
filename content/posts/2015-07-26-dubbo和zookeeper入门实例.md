---
title: Dubbo和zookeeper入门实例
author: Bridge Li
type: post
date: 2015-07-26T14:05:52+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
tags:
  - Dubbo
  - rpc
  - zookeeper
  - 分布式

---
公司项目里用到了dubbo，感觉挺好玩，以前没有玩过，自己抽时间就小小研究了一下，今天记录一下自己的学习成果。  
关于Dubbo和zookeeper是干嘛的，网上一搜一大堆所以就不多做介绍了，想了解的可以自己搜搜看，今天就只记录怎么跑一个最基本的Dubbo和zookeeper小示例程序是怎么跑起来的，当然虽然是一个demo，但和真实环境也是无差的哦。

一. 安装zookeeper

要想使用Dubbo，必须给Dubbo一个注册中心，当然这个注册中心不一定必须是zookeeper，也可以是redis等，但用zookeeper是一个相对比较好的方式，咱们暂且就这么办。  
关于zookeeper的安装，老夫窃以为这篇文章写得非常棒，就不多赘述了，大家可以直接参考：http://blog.csdn.net/wxwzy738/article/details/16330253，看了这篇文章，大家立马就会就明白怎么一回事了，看过自知。

二. Dubbo实现

Dubbo的实现肯定是要靠自己写代码啦，我的代码使用maven编译的，引入的jar文件如下：

1. 依赖的jar文件

```

<dependency>  
<groupId>junit</groupId>  
<artifactId>junit</artifactId>  
<version>4.12</version>  
<scope>test</scope>  
</dependency>

<dependency>  
<groupId>com.alibaba</groupId>  
<artifactId>dubbo</artifactId>  
<version>2.5.3</version>  
<exclusions>  
<exclusion>  
<artifactId>spring</artifactId>  
<groupId>org.springframework</groupId>  
</exclusion>  
</exclusions>  
</dependency>

<dependency>  
<groupId>org.slf4j</groupId>  
<artifactId>slf4j-log4j12</artifactId>  
<version>1.7.7</version>  
</dependency>

<dependency>  
<groupId>com.101tec</groupId>  
<artifactId>zkclient</artifactId>  
<version>0.5</version>  
</dependency>

<dependency>  
<groupId>org.springframework</groupId>  
<artifactId>spring-test</artifactId>  
<version>4.1.7.RELEASE</version>  
</dependency>

<dependency>  
<groupId>org.springframework</groupId>  
<artifactId>spring-context</artifactId>  
<version>4.1.7.RELEASE</version>  
</dependency>

```

其中junit和spring-test大家肯定看出来是干嘛的了，为了跑测试，Dubbo分为生产者和消费者，接下来我们先看看生产者是怎么实现的

2. 生产者的实现

①. 配置文件applicationProvider.xml

```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"  
xmlns:context="http://www.springframework.org/schema/context"  
xsi:schemaLocation="  
http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.0.xsd  
http://code.alibabatech.com/schema/dubbo  
http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

<!&#8211; Auto Scan &#8211;>  
<context:component-scan base-package="cn.bridgeli.provider" />

<!&#8211; Application name &#8211;>  
<dubbo:application name="hello-world-app" />

<!&#8211; registry address, used for service to register itself &#8211;>  
<dubbo:registry address="zookeeper://127.0.0.1:2181" />

<!&#8211; expose this service through dubbo protocol, through port 20880 &#8211;>  
<!&#8211;  
<dubbo:protocol name="dubbo" port="20880" />  
<dubbo:protocol name="dubbo" port="9090" server="netty"  
client="netty" codec="dubbo" serialization="hessian2" charset="UTF-8"  
threadpool="fixed" threads="100" queues="0" iothreads="9" buffer="8192"  
accepts="1000" payload="8388608" />  
&#8211;>  
<!&#8211; Service interface Concurrent Control &#8211;>  
<!&#8211; <dubbo:service interface="cn.bridgeli.provider.service.ProviderService" ref="providerService"/> &#8211;>  
<!&#8211; 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 &#8211;>  
<dubbo:annotation package="cn.bridgeli.provider.service" />  
<!&#8211; Default Protocol &#8211;>  
<!&#8211;  
<dubbo:protocol server="netty" />  
&#8211;>  
</beans>

```

注释很详细，不解释，就是dubbo和spring的集成

②. 日志

```

log4j.rootCategory=INFO, stdout, logfile, errorLog  
log4j.appender.stdout=org.apache.log4j.ConsoleAppender  
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %t %c{2}:%L &#8211; %m%n  
#log4j.category.org.springframework.beans.factory=info  
log4j.appender.consoleAppender.layout.ConversionPattern =ProcessDefinitionId=%X{mdcProcessDefinitionID}  
executionId=%X{mdcExecutionId} mdcProcessInstanceID=%X{mdcProcessInstanceID} mdcBusinessKey=%X{mdcBusinessKey} %m%n"  
log4j.appender.logfile=org.apache.log4j.DailyRollingFileAppender  
log4j.appender.logfile.file=E:/weixin/log/log.log  
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout  
log4j.appender.logfile.DatePattern=&#8217;.&#8217;yyyy-MM-dd  
#log4j.appender.logfile.layout.ConversionPattern=[%d %6p at %C.%M(%F:%L)] %m%n  
log4j.appender.logfile.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %l %m%n  
log4j.appender.logfile.Threshold=INFO  
log4j.appender.errorLog=org.apache.log4j.DailyRollingFileAppender  
log4j.appender.errorLog.file=E:/weixin/log/error.log  
log4j.appender.errorLog.layout=org.apache.log4j.PatternLayout  
log4j.appender.errorLog.DatePattern=&#8217;.&#8217;yyyy-MM-dd  
log4j.appender.errorLog.layout.ConversionPattern=[%d %6p at %C.%M(%F:%L)] %m%n  
log4j.appender.errorLog.Threshold=ERROR

```

③. 接口，也就是向外提供服务的接口ProviderService

```

package cn.bridgeli.provider.service;

public interface ProviderService {

public String yield(String data);

}

```

④. 实现类

```

package cn.bridgeli.provider.service.impl;

import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;

import cn.bridgeli.provider.service.ProviderService;

import com.alibaba.dubbo.config.annotation.Service;

@Service  
public class ProviderServiceImpl implements ProviderService {

private static final Logger LOG = LoggerFactory.getLogger(ProviderServiceImpl.class);

@Override  
public String yield(String data) {  
try {  
Thread.sleep(1000);  
} catch (InterruptedException e) {  
LOG.error("error", e);  
}  
return "Hello:" + data;  
}  
}

```

需要注意的是，这里的注解service不是spring的，而是dubbo的service，配置和接口、实现类都有了，接下来肯定是怎么把项目跑起来了，我们的测试类。

⑤. 测试类ProviderServiceTest

```

package cn.bridgeli.provider.service;

import java.io.IOException;

import org.junit.Test;  
import org.junit.runner.RunWith;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.test.context.ContextConfiguration;  
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(locations = "classpath*:applicationProvider.xml")  
public class ProviderServiceTest {

private static final Logger LOG = LoggerFactory.getLogger(ProviderServiceTest.class);

@Test  
public void testYield() {

LOG.info("Press any key to exit.");

try {  
System.in.read();  
} catch (IOException e) {  
LOG.error("error", e);  
}  
}  
}

```

这个测试类的目的就是把这个项目跑起来，没有别的目的；生产者有了，生产者有了，下面就要有消费者来消费了

3. 消费者的实现

①. 首先也是配置，日志的配置完全一样，就不多说了，所以直接看applicationConsumer.xml

```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"  
xmlns:context="http://www.springframework.org/schema/context"  
xsi:schemaLocation="  
http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.0.xsd  
http://code.alibabatech.com/schema/dubbo  
http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

<!&#8211; Auto Scan &#8211;>  
<context:component-scan base-package="cn.bridgeli.consumer" />

<!&#8211; consumer application name &#8211;>  
<dubbo:application name="consumer-of-helloworld-app" />

<!&#8211; registry address, used for consumer to discover services &#8211;>  
<dubbo:registry address="zookeeper://127.0.0.1:2181" />  
<dubbo:consumer timeout="5000"/>  
<!&#8211; which service to consume? &#8211;>  
<!&#8211; <dubbo:reference id="providerService" interface="cn.bridgeli.provider.service.ProviderService" /> &#8211;>  
<!&#8211; 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 &#8211;>  
<dubbo:annotation package="cn.bridgeli.consumer" />  
</beans>

```

②. 消费者的接口ConsumerService

```

package cn.bridgeli.consumer.service;

public interface ConsumerService {

public String consume(String str);

}

```

③. 接口的实现

```

package cn.bridgeli.consumer.service.impl;

import org.springframework.stereotype.Service;

import cn.bridgeli.consumer.service.ConsumerService;  
import cn.bridgeli.provider.service.ProviderService;

import com.alibaba.dubbo.config.annotation.Reference;

@Service  
public class ConsumerServiceImpl implements ConsumerService {

@Reference  
private ProviderService providerService;

@Override  
public String consume(String str) {

String hello = providerService.yield(str);  
return hello;

}

}

```

也就是在视线里面调用了生产者里面的方法，以前我们都是用http这次dubbo直接帮我们实现了，省心多了吧？但是我相信大家看了这段代码会发现一个问题，我们消费者中需要引入生产者的接口和实体类，所以在消费者的pom文件中增加：

```

<dependency>  
<groupId>cn.bridgeli</groupId>  
<artifactId>provider</artifactId>  
<version>0.0.1-SNAPSHOT</version>  
</dependency>

```

至于怎么打包生产者的接口（注意仅仅是接口，不需要实现）和实体类，就不用我多说了，借助于eclipse大家都会，打成jar文件之后，怎么上传到本地私服，老夫以前的博文有一篇写过，[就是这篇][1]，大家可以看看，当然这是命令行的，大家也可以借助于nexus的web界面导入，很简单的。  
好了一切都准备妥当之后，当然是跑起来啦，依然借助于junit。

④. ConsumerServiceTest

```

package cn.bridgeli.consumer.service;

import javax.annotation.Resource;

import org.junit.Assert;  
import org.junit.Test;  
import org.junit.runner.RunWith;  
import org.springframework.test.context.ContextConfiguration;  
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(locations = "classpath*:applicationConsumer.xml")  
public class ConsumerServiceTest {

@Resource  
private ConsumerService consumerService;

@Test  
public void testConsume() {  
String str = "bridge";  
String consume = consumerService.consume(str);  
String result = "Hello:" + str;  
Assert.assertTrue(result.equals(consume));  
}  
}

```

好了，就这样我们最简单的一个dubbo和zookeeper的demo就完成了，大家可以尽可能的去体会dubbo给我们带来的方便和分布式的体验了

原文代码下载地址：[dubbo demo][2]

 [1]: https://www.bridgeli.cn/archives/94 "怎么在maven项目中引用本地Java类库"
 [2]: https://github.com/bridgeli/dubbo "dubbo demo"