---
title: 如何构建一个可重复读流 InputStream 的 HttpServletRequest？
author: Bridge Li
type: post
date: 2024-05-03T11:48:02+00:00

categories:
  - Java
tags:
  - InputStream
  - 可重复读流

---
之前在某公司工作的时候，领导要求所有前端向后端传递的参数都要经过前端加密，后端解密。说一句题外话：个人认为这种操作纯属脱裤子放屁，没啥用。因为前端代码都是公开的，无论你采用对称加密、非对称加密，或者摘要算法验签等等，对于稍懂技术的人来说，稍稍分析一下就能找到前端加密的方法，然后直接用相同的方式加密就行，所以这就是障眼法，只能骗骗不懂技术的人。不过领导的要求吗，既然定下来了，那么我们总要服从。因为每个方法都需要有这个解密或者验签的过程，我们自然而然想要到了通过 Filter、Interceptor 或者 AOP 等技术统一来做，不可能在各个方法中做这件事，在但是我们都知道，对于 post、put 等请求，参数都是放在请求体中的，需要通过流读出来，而流是不可以重复读的，所以我们应该怎么来解决这个问题，来构造一个可以重复读流 InputStream 的 HttpServletRequest。

解决方法：使用自定义类来缓存 stream 即可 RequestWrapper 类：缓存字节数据

```

package cn.bridgeli.filter;

import cn.bridgeli.utils.http.HttpHelper;

import javax.servlet.ReadListener;  
import javax.servlet.ServletInputStream;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletRequestWrapper;  
import java.io.BufferedReader;  
import java.io.ByteArrayInputStream;  
import java.io.IOException;  
import java.io.InputStreamReader;  
import java.nio.charset.StandardCharsets;

/**  
* 构建可重复读取inputStream的request  
*  
* @author BridgeLi  
*/  
public class RepeatedlyRequestWrapper extends HttpServletRequestWrapper {  
private final byte[] body;

public RepeatedlyRequestWrapper(HttpServletRequest request) {  
super(request);  
body = HttpHelper.getBodyString(request).getBytes(StandardCharsets.UTF_8);  
}

@Override  
public BufferedReader getReader() throws IOException {  
return new BufferedReader(new InputStreamReader(getInputStream()));  
}

@Override  
public ServletInputStream getInputStream() throws IOException {  
final ByteArrayInputStream bais = new ByteArrayInputStream(body);  
return new ServletInputStream() {  
@Override  
public int read() throws IOException {  
return bais.read();  
}

@Override  
public int available() throws IOException {  
return body.length;  
}

@Override  
public boolean isFinished() {  
return false;  
}

@Override  
public boolean isReady() {  
return false;  
}

@Override  
public void setReadListener(ReadListener readListener) {

}  
};  
}  
}

package cn.bridgeli.utils.http;

import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;

import javax.servlet.ServletRequest;  
import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStream;  
import java.io.InputStreamReader;  
import java.nio.charset.StandardCharsets;

/**  
* 通用http工具封装  
*  
* @author BridgeLi  
*/  
public class HttpHelper {  
private static final Logger LOGGER = LoggerFactory.getLogger(HttpHelper.class);

public static String getBodyString(ServletRequest request) {  
StringBuilder sb = new StringBuilder();  
try (InputStream inputStream = request.getInputStream()) {  
BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, StandardCharsets.UTF_8));  
String line = "";  
while ((line = reader.readLine()) != null) {  
sb.append(line);  
}  
} catch (IOException e) {  
LOGGER.error("getBodyString出现问题！", e);  
}  
return sb.toString();  
}  
}

```

然后，可以在 Servlet 或 Filter 中使用 RepeatableFilter 替换原始的 HttpServletRequest。

```

package cn.bridgeli.filter;

import cn.bridgeli.filter.RepeatedlyRequestWrapper;

import javax.servlet.Filter;  
import javax.servlet.FilterChain;  
import javax.servlet.FilterConfig;  
import javax.servlet.ServletException;  
import javax.servlet.ServletRequest;  
import javax.servlet.ServletResponse;  
import javax.servlet.http.HttpServletRequest;  
import java.io.IOException;

/**  
* Repeatable 过滤器  
*  
* @author BridgeLi  
*/  
public class RepeatableFilter implements Filter {  
@Override  
public void init(FilterConfig filterConfig) throws ServletException {

}

@Override  
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  
throws IOException, ServletException {  
ServletRequest requestWrapper = null;  
if (request instanceof HttpServletRequest) {  
requestWrapper = new RepeatedlyRequestWrapper((HttpServletRequest) request);  
}  
if (null == requestWrapper) {  
chain.doFilter(request, response);  
} else {  
chain.doFilter(requestWrapper, response);  
}  
}

@Override  
public void destroy() {

}  
}

```

这样就解决了。

其实，对于可重复读流 InputStream 的 HttpServletRequest，除了我前公司领导的这个不合理需求，但是在实际工作中还是很有意义的，例如统一记录日志打一下参数，或者通过 AOP 做一些其他的业务等等。

最后，我在前公司的时候那时候还没有 ChatGPT，但是在 2022 年的 11 月底，ChatGPT 横空出世，现在把这个需求贴到 ChatGPT 中，ChatGPT 哗哗的就能解决了，方式方法一样。