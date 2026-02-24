---
title: Junit Test之Easy Mock Test入门
author: Bridge Li
type: post
date: 2016-02-29T14:37:48+00:00

duoshuo_thread_id:
  - 6.2567222569075E+18
categories:
  - Java
tags:
  - Easy Mock
  - Junit

---
这一段时间公司的项目进行分模块分层进行专人维护开发，所以就会有不同的service和dao有不同的人来开发，这里我们假设service和dao不同的人开发，service是依赖dao的，如果我们的dao开发人员比较忙并没有把dao模块开发好，service如果要对自己的模块进行测试该怎么做呢？这个时候我们的Easy Mock Test就可以派上用场了。  
首先开发service的和dao的会讨论商量出来一套接口，假设dao的接口如下：

```

package cn.bridgeli.dao;

import cn.bridgeli.model.User;

public interface UserDao {  
public User getUserById(int id);  
public User getUserByUsername(String username);  
//&#8230;  
}

```

dao模块的小朋友把它打成一个jar包，扔给service开发人员，然后我们亲爱的service开发人员就自己玩去了，最后我们的service开发人员完成了自己的任务，写下了如下的代码，当然这只是一个demo而已：

```

package cn.bridgeli.service.impl;

import cn.bridgeli.dao.UserDao;  
import cn.bridgeli.model.User;  
import cn.bridgeli.service.UserService;

public class UserServiceImpl implements UserService {

private UserDao userDao;

public void setUserDao(UserDao userDao) {  
this.userDao = userDao;  
}

@Override  
public boolean login(String username, String password) {  
User user = userDao.getUserByUsername(username);  
System.out.println("id==" + user.getId());  
if (user != null) {  
String passwordInDao = user.getPassword();  
if (passwordInDao != null && passwordInDao.equalsIgnoreCase(password)) {  
return true;  
}  
}  
return false;  
}

@Override  
public User getUserById(int userId) {  
return userDao.getUserById(userId);

}

}

```

但是我们的service怎么知道自己写的有没有问题呢？现在我们的service想对自己的模块进行测试，但dao开发人员还没开始，这肯定是没办法开始的，那么怎么办呢？很简单，只需要这么做就可以了：

```

package cn.bridgeli.service;

import org.easymock.EasyMock;  
import org.junit.Assert;  
import org.junit.Test;

import cn.bridgeli.dao.UserDao;  
import cn.bridgeli.model.User;  
import cn.bridgeli.service.impl.UserServiceImpl;

public class UserServiceTest {

//@Test(expected = RuntimeException.class)  
@Test  
public void testLogin() {  
String userName = "bridgeli";  
String password = "abc123_";

//1、创建mock对象，以接口形式创建  
UserDao userDao= EasyMock.createMock(UserDao.class);

//2、设定参预期和返回，查询预期值得到所设定的预期结果  
User user = new User();  
user.setId(1);  
user.setUserName("bridgeli");  
user.setPassword("abc123_");  
//&#8230;

EasyMock.expect(userDao.getUserByUsername("bridgeli")).andReturn(user).times(1);  
// userDao.getUserById(1);  
// EasyMock.expectLastCall().andReturn(user);  
//  
userDao.getUserByUserName(userName);  
EasyMock.expectLastCall();

//3、结束录制  
EasyMock.replay(userDao);

UserService userService = new UserServiceImpl();  
((UserServiceImpl)userService).setUserDao(userDao);

boolean loginResult = userService.login(userName, password);  
Assert.assertTrue(loginResult);

//User userIdDao = userService.getUserById(1);  
//Assert.assertNotNull(userIdDao);

//4、回放录制  
EasyMock.verify(userDao);

}  
}

```

这就要求我们引入EasyMock的类库，pom文件如下：

```

<dependency>  
<groupId>org.easymock</groupId>  
<artifactId>easymock</artifactId>  
<version>3.4</version>  
</dependency>

```

现在我们直接跑这个test是可以直接跑的，需要说明的是：

1. EasyMock.expect(userDao.getUserByUsername(&#8220;bridgeli&#8221;))  
.andReturn(user).times(1) 是指我们要调用dao中的getUserByUsername时，返回user对象。  
2. times以为调用的次数，默认就是1，否则调几次就是写几。  
3. 我们给service和传的参数是bridgeli，所以返回的user对象和我们定义的一样，测试通过，但如果我们传的参数是其他的肯定就不行了，那么如果我们不希望是固定参数呢？这时候我们可以这么做，把：

```

EasyMock.expect(userDao.getUserByUsername("bridgeli")).andReturn(user).times(1)

```

改成：

```

EasyMock.expect(userDao.getUserByUsername(EasyMock.isA(String.class))).andReturn(user).times(1)

```

就好了，除此之外还有一些常用的方法：

anyInt()，anyObject()，isNull()，same()，startsWith()等等

4. 期待方法返回异常，可以这么做（面向异常编程）：

```

EasyMock.expect(userDao.getUserByUsername("bridgeli")).andThrow(new RuntimeException()).times(1)

```

具体还有很多方法，但因为这是一个入门教程，所以师傅领进门，修行在个人，就留给读者大胆的去测试EasyMock里面的各种各样的方法了。

最后简单说一下Mock测试的使用场景：

1. 真实对象具有不可确定的行为（产生不可预测的结果，如股票行为，如果某种情况发生的概率极其小，可以导致无法测试）；  
2. 真实对象很难被创建（如request和response对象，和容器相关，我们不能自己new，这个时候就可以mock）；  
3. 真实对象的某些行为很难触发（如网络错误）；  
4. 真实对象令程序的运行速度很慢；  
4. 真实对象有（或者是）用户界面；  
5. 测试需要询问真实对象他是如何被调用的（例如：测试可能需要验证某个回调函数是否被调用了）；  
6. 等等，读者可以去网上自己找一下mock测试的资料。