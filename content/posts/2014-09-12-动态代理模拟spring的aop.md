---
title: 动态代理模拟Spring的AOP
author: Bridge Li
type: post
date: 2014-09-12T13:21:07+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
---
这两天研究了一下Java的动态代理，自己闲着无聊，用动态代理模拟了一下Spring的AOP，代码如下，当然真正的Spring是直接操作二进制文件，很复杂，有兴趣的可以自己研究下。

```

package cn.bridgeli.aop;

public interface UserService {
    void addUser();
}

```

```

package cn.bridgeli.aop;

public class UserServiceImpl implements UserService {

    public void addUser() {
        System.out.println("User add&#8230;");
    }

}

```

```

package cn.bridgeli.aop;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class UserServiceProxy implements InvocationHandler {

    private UserService userService;

    public UserServiceProxy(UserService userService) {
        this.userService = userService;
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("User add start&#8230;");
        Object object = method.invoke(userService, args);
        System.out.println("User add end&#8230;");
        return object;
    }

}

```

```

package cn.bridgeli.aop;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

import org.junit.Test;

public class UserServiceTest {
    @Test
    public void testAddUser() {
        UserService userService = new UserServiceImpl();
        InvocationHandler invocationHandler = new UserServiceProxy(userService);
        UserService userServiceProxy = (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(), userService.getClass().getInterfaces(), invocationHandler);
        userServiceProxy.addUser();
    }
}

```