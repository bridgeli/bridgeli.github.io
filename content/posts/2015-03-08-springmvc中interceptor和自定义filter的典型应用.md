---
title: SpringMVC中Interceptor和自定义filter的典型应用
author: Bridge Li
type: post
date: 2015-03-08T15:03:43+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
tags:
  - filter
  - interceptor
  - spring mvc

---
今天写写老夫最擅长的Java web，在Java web中Interceptor和filter应用十分广泛，今天就写一个在我们的项目中的一个最基本的应用，过滤或者拦截未登录用户访问某些资源。

1. SpringMVC中Interceptor  
SpringMVC 中的Interceptor 拦截器是相当重要和相当有用的，它的主要作用是拦截用户的请求并进行相应的处理。比如通过它来进行权限验证，或者是来判断用户是否登陆等等。今天就写一个Interceptor在开发中的典型应用：某一系统某些方法肯定是需要用户登陆才能访问的，而另外一些肯定不需要用户登陆就能访问（这样的例子很多，老夫就不举例说明了），那么我们怎么做，才能做到呢？这个时候Interceptor就派上用场了，下面是一个小例子，供参考：  
spring-servlet.xml核心代码如下：

```  
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context"  
xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-4.0.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-4.0.xsd  
http://www.springframework.org/schema/mvc  
http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">

<mvc:interceptors>  
<bean id="permissionInterceptor" class="cn.bridgeli.demo.interceptor.PermissionInterceptor"></bean>  
</mvc:interceptors>

&#8230;&#8230;

</beans>  
```

对应的Interceptor的实现：

```  
package cn.bridgeli.demo.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.ModelAndViewDefiningException;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
import org.springframework.web.util.UrlPathHelper;

import cn.bridgeli.demo.entity.User;

public class PermissionInterceptor extends HandlerInterceptorAdapter {

    private UrlPathHelper urlPathHelper = new UrlPathHelper();

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        User user = (User) request.getSession().getAttribute("USER");
        String url = urlPathHelper.getLookupPathForRequest(request);
        int flag = url.indexOf("/admin/");
        if (user == null && flag != -1) {
            ModelAndView mav = new ModelAndView("error/permissionerror");
            mav.addObject("ERRORMSG", "对不起，您没有登录，无法使用该功能！");
            throw new ModelAndViewDefiningException(mav);
        }
        return true;
    }

}  
```

关于InterceptorAdapter的更多用法，大家可以参考http://haohaoxuexi.iteye.com/blog/1750680，老夫以为这篇文章说的相对比较详细易懂，除此之外，我们还可以通过自定义filter来实现；

2. 自定义filter  
filter代码：  
```  
package cn.bridgeli.demo.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import cn.bridgeli.demo.entity.User;

public class SessionFilter implements Filter {

    public SessionFilter() {
    }

    public void destroy() {
    }

    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;

        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("USER");

        if (-1 != request.getRequestURI().indexOf("/admin/") && null == user) {
            request.getRequestDispatcher("/WEB-INF/admin/login.jsp").forward(request, response);
            return;
        } else {
            chain.doFilter(request, response);
        }
    }

}  
```

对应的在web.xml中的配置：

```  
<filter>  
<description>  
</description>  
<display-name>SessionFilter</display-name>  
<filter-name>SessionFilter</filter-name>  
<filter-class>cn.bridgeli.demo.filter.SessionFilter</filter-class>  
</filter>  
<filter-mapping>  
<filter-name>SessionFilter</filter-name>  
<url-pattern>*.do</url-pattern>  
</filter-mapping>  
```

其实在开发中Interceptor和filter应用十分广泛，他可以对用户的请求做各种过滤和拦截，例如我们还可以做一个EncodingFilter，对我们所有的请求进行过滤，来解决乱码问题，因为核心代码就两句话：

```  
request.setCharacterEncoding("utf-8");  
chain.doFilter(request, response);  
```

所以就不在多说了，大家可以自己探讨更多用法