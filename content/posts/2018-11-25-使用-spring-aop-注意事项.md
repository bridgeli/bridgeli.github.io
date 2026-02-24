---
title: 使用 Spring AOP 注意事项
author: Bridge Li
type: post
date: 2018-11-25T11:48:23+00:00

categories:
  - Java
tags:
  - spring aop
---
说实话，由于我个人某些基础不是很牢固，所以前一段时间关于 Spring Aop 踩了一个坑，其实很简单，今天就记录一下，先说结论：

不能被 Spring AOP 增强的方法：

1. 基于接口的动态代理：除 public 外的其它所有的方法，此外 public static 也不能被增强  
2. 基于 CGLib 的动态代理：private、static、final 的方法，也就是只有 public 和 protected 可以，但是要注意切入点语法的配置

测试用例如下，pom 文件：

```

<dependency>  
<groupId>org.slf4j</groupId>  
<artifactId>slf4j-log4j12</artifactId>  
<version>1.7.7</version>  
</dependency>  
<dependency>  
<groupId>org.springframework</groupId>  
<artifactId>spring-context</artifactId>  
<version>4.3.11.RELEASE</version>  
<scope>test</scope>  
</dependency>  
<dependency>  
<groupId>org.aspectj</groupId>  
<artifactId>aspectjweaver</artifactId>  
<version>1.8.10</version>  
</dependency>  
<dependency>  
<groupId>org.springframework</groupId>  
<artifactId>spring-context</artifactId>  
<version>4.3.11.RELEASE</version>  
</dependency>

```

配置文件就比较简单了：

```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xmlns:context="http://www.springframework.org/schema/context"  
xmlns:aop="http://www.springframework.org/schema/aop"  
xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-2.5.xsd  
http://www.springframework.org/schema/aop  
http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">  
<context:annotation-config/>  
<context:component-scan base-package="cn.bridgeli"/>  
<aop:aspectj-autoproxy/>

</beans>

```

切面类：

```

package cn.bridgeli.demo.aop;

import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Before;  
import org.aspectj.lang.annotation.Pointcut;  
import org.springframework.stereotype.Component;

@Aspect  
@Component  
public class LogInterceptor {  
// @Pointcut("execution(\* \* com.bjsxt.service..*.add(..))")  
@Pointcut("execution(\* cn.bridgeli.demo.service..\*.*(..))")  
public void myMethod() {  
}

@Before("myMethod()")  
public void before() {  
System.out.println("method before");  
}

@Around("myMethod()")  
public void aroundMethod(ProceedingJoinPoint pjp) throws Throwable {  
System.out.println("method around start");  
pjp.proceed();  
System.out.println("method around end");  
}

}

```

测试类：

```

package cn.bridgeli.demo.service;

import org.springframework.context.support.ClassPathXmlApplicationContext;  
import org.springframework.stereotype.Service;

@Service("userService")  
public class UserService {

/**  
* private方法因为修饰符访问权限的控制，无法被子类覆盖  
*/  
private void method1() {  
System.out.println("method1 executed");  
}

/**  
* final 方法无法被子类覆盖  
*/  
private final void method2() {  
System.out.println("method2 executed");  
}

/**  
* static 方法是类级别的方法，无法被子类覆盖  
*/  
private static void method3() {  
System.out.println("method3 executed");  
}

/**  
* public 方法可以被子类覆盖，因此可以被动态字节码增强  
*/  
public void method4() {  
System.out.println("method4 executed");  
}

/**  
* final 方法无法被子类覆盖  
*/  
public final void method5() {  
System.out.println("method5 executed");  
}

/**  
* protected 方法可以被子类覆盖，因此可以被动态字节码增强  
*/  
protected void method6() {  
System.out.println("method6 executed");  
}

/**  
* 测试  
*  
* @param args  
*/  
public static void main(String[] args) {

ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");  
UserService userService = (UserService) ctx.getBean("userService");  
System.out.println("initContext successfully");

System.out.println("before method1");  
userService.method1();  
System.out.println("after1 method1");

System.out.println("before method2");  
userService.method2();  
System.out.println("after1 method2");

System.out.println("before method3");  
method3();  
System.out.println("after1 method3");

System.out.println("before method4");  
userService.method4();  
System.out.println("after1 method4");

System.out.println("before method5");  
userService.method5();  
System.out.println("after1 method5");

System.out.println("before method6");  
userService.method6();  
System.out.println("after1 method6");

if (ctx != null) {  
ctx.close();  
}  
System.out.println("close context successfully");  
}  
}

```

参考资料：  
1. http://jinnianshilongnian.iteye.com/blog/1857189  
2. https://blog.csdn.net/yangshangwei/article/details/78094196