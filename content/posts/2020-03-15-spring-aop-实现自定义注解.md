---
title: Spring AOP 实现自定义注解
author: Bridge Li
type: post
date: 2020-03-15T07:32:50+00:00

categories:
  - Java
tags:
  - annotation
  - AOP
  - Aspect
  - 切面
  - 日志
  - 注解
  - 自定义注解
  - 请求拦截

---
自工作后，除了一些小项目配置事务使用过 AOP，真正自己写 AOP 机会很少，另一方面在工作后还没有写过自定义注解，一直很好奇注解是怎么实现他想要的功能的，刚好做项目的时候，经常有人日志打得不够全，经常出现问题了，查日志的才发现忘记打了，所以趁此机会，搜了一些资料，用 AOP + 自定义注解，实现请求拦截，自定义打日志，玩一下这两个东西，以下是自己完的一个小例子，也供需要的同学参考。

1. 注解如下：

```

package cn.bridgeli.demo.annotation;

import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;

/**  
* @author bridgeli  
*/  
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface MyLog {  
/**  
* 方法描述  
*  
* @return  
*/  
String desc() default "";  
}

```

2. 切面

```

package cn.bridgeli.demo.annotation;

import cn.bridgeli.utils.AuthorizeUtil;  
import cn.bridgeli.entity.Principal;  
import lombok.extern.slf4j.Slf4j;  
import org.apache.commons.lang3.StringUtils;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Pointcut;  
import org.springframework.stereotype.Component;

/**  
* @author bridgeli  
* 1. 这是一个切面类  
*/  
@Aspect  
@Component  
@Slf4j  
public class MyLogAspect {

/**  
* 2. PointCut表示这是一个切点，@annotation表示这个切点切到一个注解上，后面带该注解的全类名  
* 切面最主要的就是切点，所有的故事都围绕切点发生  
* logPointCut()代表切点名称  
*/  
@Pointcut("@annotation(cn.bridgeli.demo.annotation.MyLog)")  
public void logPointCut() {  
}

/**  
* 3. 环绕通知  
*  
* @param joinPoint  
* @param myLog  
* @return  
*/  
@Around(value = "logPointCut() && @annotation(myLog)", argNames = "joinPoint,myLog")  
public Object logAround(ProceedingJoinPoint joinPoint, MyLog myLog) {  
// 获取方法名  
String methodFullPathName = joinPoint.getTarget().getClass().getName() + "#" + joinPoint.getSignature().getName();

// 获取参数  
String params = StringUtils.join(joinPoint.getArgs(), ";");

Principal currentUser = AuthorizeUtil.getCurrentUser();  
log.info("当前登陆用户：" + (null == currentUser ? "" : currentUser.toString()) + "，进入 [ " + methodFullPathName + " ] 方法, 方法的描述：" + myLog.desc() + "，参数为:" + params);

// 继续执行方法  
long startTime = System.currentTimeMillis();  
Object result = null;  
try {  
result = joinPoint.proceed();  
} catch (Throwable e) {  
log.error("切面执行报错，参数：" + params, e);  
}  
long elapsed = System.currentTimeMillis() &#8211; startTime;

log.info("[ " + methodFullPathName + " ] 方法执行结束，返回值为：" + (null == result ? "" : result.toString()) + "，用时：" + elapsed);

return result;  
}  
}

```

然后只需要在想使用的地方 @MyLog 就可以了，当然也可以加上 @MyLog(desc = &#8220;这是方法描述&#8221;)，这样打出来的日志还会有方法是做什么的，别人看日志的时候能够一目了然。

需要说明的是，我在写这个切面的时候遇到的一个小问题，在网上看 AOP 的注解，很多人在举例子的时候都是不关注 @Around 的返回值，所以方法的返回值都写的 void，因为我对 AOP 也不是很熟，所以当时同样写了一个 void，结果写好一测试，返回拦截也正常，日志也打印了，被拦截的方法执行也挺正常，但是就是没有了返回值，当时还很奇怪，然后随便试了下返回值改成 Object，竟然对了，所以这是一个小坑，也是很多人没有说明的一点，大家可以注意下，其实这个问题也很容易想到，@Around 是环绕拦截，在执行完被拦截的方法之后，会继续执行切面方法，如果切面方法没有返回值，那么自然而然就没有返回值了，同理 @After 拦截个人猜测也应该有同样的问题，大家可以测试下。

最后再说一个小问题，大家应该可以看到在切面方法里面，我没有使用 StringBuilder，而是直接用了“+”，个人曾经看过一些资料，JDK 的 JIT 技术和编译器优化，会把“+”优化成 StringBuilder（对循环无效，还需要自己老老实实的写），所以挑简单的写了。