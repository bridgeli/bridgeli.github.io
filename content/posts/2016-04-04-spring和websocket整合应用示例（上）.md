---
title: Spring和websocket整合应用示例（上）
author: Bridge Li
type: post
date: 2016-04-04T14:05:19+00:00

duoshuo_thread_id:
  - 6.2697018663601E+18
categories:
  - Java
tags:
  - 长连接
---
嗯，这次真的仅仅是一个入门教程，因为老夫表示自己也不会。近期老夫参与开发公司的一个CRM系统，系统中有很多消息的推送，由一个同事负责，其用到了websocket技术，老夫比较感兴趣，删繁就简，整理了一个教程，留作自己笔记，因很多原理老夫也是不甚了了，以备将来用到了有资料可查。

1. maven依赖

```

<dependency>  
<groupId>javax.servlet</groupId>  
<artifactId>javax.servlet-api</artifactId>  
<version>3.1.0</version>  
</dependency>  
<dependency>  
<groupId>com.fasterxml.jackson.core</groupId>  
<artifactId>jackson-core</artifactId>  
<version>2.3.0</version>  
</dependency>  
<dependency>  
<groupId>com.fasterxml.jackson.core</groupId>  
<artifactId>jackson-databind</artifactId>  
<version>2.3.0</version>  
</dependency>  
<dependency>  
<groupId>org.springframework</groupId>  
<artifactId>spring-websocket</artifactId>  
<version>4.0.1.RELEASE</version>  
</dependency>  
<dependency>  
<groupId>org.springframework</groupId>  
<artifactId>spring-messaging</artifactId>  
<version>4.0.1.RELEASE</version>  
</dependency>

```

2. spring-servlet的配置

```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
xmlns:context="http://www.springframework.org/schema/context"  
xmlns:mvc="http://www.springframework.org/schema/mvc"  
xmlns:tx="http://www.springframework.org/schema/tx" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xmlns:websocket="http://www.springframework.org/schema/websocket"  
xsi:schemaLocation="  
http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.1.xsd  
http://www.springframework.org/schema/mvc  
http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd  
http://www.springframework.org/schema/tx  
http://www.springframework.org/schema/tx/spring-tx-3.1.xsd  
http://www.springframework.org/schema/websocket  
http://www.springframework.org/schema/websocket/spring-websocket.xsd">

&#8230;&#8230;

<!&#8211; websocket &#8211;>  
<bean id="websocket" class="cn.bridgeli.websocket.WebsocketEndPoint"/>  
<websocket:handlers>  
<websocket:mapping path="/websocket" handler="websocket"/>  
<websocket:handshake-interceptors>  
<bean class="cn.bridgeli.websocket.HandshakeInterceptor"/>  
</websocket:handshake-interceptors>  
</websocket:handlers>  
</beans>

```

其中，path对应的路径就是前段通过ws协议调的接口路径

3. HandshakeInterceptor的实现

```

package cn.bridgeli.websocket;

import cn.bridgeli.utils.UserManager;  
import cn.bridgeli.util.DateUtil;  
import cn.bridgeli.sharesession.UserInfo;  
import org.apache.commons.lang.StringUtils;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.http.server.ServerHttpRequest;  
import org.springframework.http.server.ServerHttpResponse;  
import org.springframework.web.context.request.RequestContextHolder;  
import org.springframework.web.context.request.ServletRequestAttributes;  
import org.springframework.web.socket.WebSocketHandler;  
import org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor;

import java.util.Date;  
import java.util.Map;

/**  
* @Description :创建握手（handshake）接口  
* @Date : 16-3-3  
*/

public class HandshakeInterceptor extends HttpSessionHandshakeInterceptor{  
private static final Logger logger = LoggerFactory.getLogger(HandshakeInterceptor.class);

@Override  
public boolean beforeHandshake(ServerHttpRequest request,  
ServerHttpResponse response, WebSocketHandler wsHandler,  
Map<String, Object> attributes) throws Exception {  
logger.info("建立握手前&#8230;");  
ServletRequestAttributes attrs = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();  
UserInfo currUser = UserManager.getSessionUser(attrs.getRequest());  
UserSocketVo userSocketVo = new UserSocketVo();  
String email= "";

if(null != currUser){  
email = currUser.getEmail();  
}  
if(StringUtils.isBlank(email)){  
email = DateUtil.date2String(new Date());  
}  
userSocketVo.setUserEmail(email);  
attributes.put("SESSION_USER", userSocketVo);

return super.beforeHandshake(request, response, wsHandler, attributes);  
}

@Override  
public void afterHandshake(ServerHttpRequest request,  
ServerHttpResponse response, WebSocketHandler wsHandler,  
Exception ex) {  
logger.info("建立握手后&#8230;");  
super.afterHandshake(request, response, wsHandler, ex);  
}  
}

```

因为老夫不是很懂，所以最大限度的保留原代码，这其实就是从单点登录中取出当前登录用户，转成UserSocketVo对象，放到Map中。所以接下来我们看看UserSocketVo对象的定义

4. UserSocketVo的定义

```

package cn.bridgeli.websocket;

import org.springframework.web.socket.WebSocketSession;

import java.util.Date;

/**  
* @Description : 用户socket连接实体  
* @Date : 16-3-7  
*/  
public class UserSocketVo {

private String userEmail; //用户邮箱  
private Date connectionTime; //成功连接时间  
private Date preRequestTime; //上次请求时间  
private Date newRequestTime; //新请求时间  
private Date lastSendTime = new Date(); //下架消息最近一次发送时间  
private Date lastTaskSendTime = new Date(); //待处理任务最近一次发送时间  
private WebSocketSession webSocketSession; //用户对应的wsSession 默认仅缓存一个

// getXX and setXX  
}

```

其中最重要的就是这个WebSocketSession这个属性了，后面我们要用到

5. WebsocketEndPoint的实现

```

package cn.bridgeli.websocket;

import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.web.socket.CloseStatus;  
import org.springframework.web.socket.TextMessage;  
import org.springframework.web.socket.WebSocketSession;  
import org.springframework.web.socket.handler.TextWebSocketHandler;

/**  
* @Description : websocket处理类  
* @Date : 16-3-3  
*/

public class WebsocketEndPoint extends TextWebSocketHandler{  
private static final Logger logger = LoggerFactory.getLogger(WebsocketEndPoint.class);

@Autowired  
private NewsListenerImpl newsListener;

@Override  
protected void handleTextMessage(WebSocketSession session,  
TextMessage message) throws Exception {  
super.handleTextMessage(session, message);  
TextMessage returnMessage = new TextMessage(message.getPayload()+" received at server");  
session.sendMessage(returnMessage);  
}

/**  
* @Description :　建立连接后  
* @param session  
* @throws Exception  
*/  
@Override  
public void afterConnectionEstablished(WebSocketSession session) throws Exception{  
UserSocketVo userSocketVo = (UserSocketVo)session.getAttributes().get("SESSION_USER");  
if(null != userSocketVo){  
userSocketVo.setWebSocketSession(session);  
if(WSSessionLocalCache.exists(userSocketVo.getUserEmail())){  
WSSessionLocalCache.remove(userSocketVo.getUserEmail());  
}  
WSSessionLocalCache.put(userSocketVo.getUserEmail(), userSocketVo);  
newsListener.afterConnectionEstablished(userSocketVo.getUserEmail());  
}  
logger.info("socket成功建立连接&#8230;");  
super.afterConnectionEstablished(session);  
}

@Override  
public void afterConnectionClosed(WebSocketSession session,CloseStatus status) throws Exception{  
UserSocketVo userSocketVo = (UserSocketVo)session.getAttributes().get("SESSION_USER");  
if(null != userSocketVo){  
WSSessionLocalCache.remove(userSocketVo.getUserEmail());  
}  
logger.info("socket成功关闭连接&#8230;");  
super.afterConnectionClosed(session, status);  
}  
}

```

6. WSSessionLocalCache的实现

```

package cn.bridgeli.websocket;

import java.io.Serializable;  
import java.util.ArrayList;  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;

/**  
* @Description :本地缓存WebSocketSession实例  
* @Date : 16-3-7  
*/  
public class WSSessionLocalCache implements Serializable {

private static Map<String, UserSocketVo> wsSessionCache = new HashMap<>();

public static boolean exists(String userEmail){  
if(!wsSessionCache.containsKey(userEmail)){  
return false;  
}else{  
return true;  
}  
}

public static void put(String userEmail, UserSocketVo UserSocketVo){  
wsSessionCache.put(userEmail, UserSocketVo);  
}

public static UserSocketVo get(String userEmail){  
return wsSessionCache.get(userEmail);  
}

public static void remove(String userEmail){  
wsSessionCache.remove(userEmail);  
}

public static List<UserSocketVo> getAllSessions(){

return new ArrayList<>(wsSessionCache.values());  
}  
}

```

看了其实现，作用就比较明显了吧，存放每个UserSocketVo的最新数据，其实到这里我们websocket的实现已经算完了，但还有一个核心类（关于业务逻辑查理的类）没有实现，下篇我们就看怎么实现这个类