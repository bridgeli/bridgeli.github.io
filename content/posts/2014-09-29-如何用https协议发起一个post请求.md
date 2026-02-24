---
title: 如何用https协议发起一个post请求
author: Bridge Li
type: post
date: 2014-09-29T12:57:06+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Java
---
这两天研究了微信公众号的开发，发现微信做的太好了，前景太可怕了，如果按照这个趋势，那么将来手机上也许只装一个微信客户端也许就可以做任何事了，其中微信公众号开发自定义菜单时，微信要求用https协议post到微信服务器一个JSON字符串，这里面有两个难点：1. https协议，2. 如何post数据到微信服务器。刚好csdn博主柳峰，有一篇文章是讲解这个的，所以就拿来参考一下，具体代码如下：

```

import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

import javax.net.ssl.X509TrustManager;

public class MyX509TrustManager implements X509TrustManager {

    @Override
    public void checkClientTrusted(X509Certificate[] arg0, String arg1) throws CertificateException {

    }

    @Override
    public void checkServerTrusted(X509Certificate[] arg0, String arg1) throws CertificateException {

    }

    @Override
    public X509Certificate[] getAcceptedIssuers() {
        return null;
    }

}

```

这个是说，相信所有的安全证书，无论是不是安全机构颁发

```

public static JSONObject httpsRequest(String requestUrl, String requestMethod, String outputStr) {
    JSONObject jsonObject = null;
    StringBuffer buffer = new StringBuffer();
    try {
// 创建SSLContext对象，并使用我们指定的信任管理器初始化  
        TrustManager[] tm = { new MyX509TrustManager() };
        SSLContext sslContext = SSLContext.getInstance("SSL", "SunJSSE");
        sslContext.init(null, tm, new java.security.SecureRandom());
// 从上述SSLContext对象中得到SSLSocketFactory对象  
        SSLSocketFactory ssf = sslContext.getSocketFactory();

        URL url = new URL(requestUrl);
        HttpsURLConnection httpUrlConn = (HttpsURLConnection) url.openConnection();
        httpUrlConn.setSSLSocketFactory(ssf);

        httpUrlConn.setDoOutput(true);
        httpUrlConn.setDoInput(true);
        httpUrlConn.setUseCaches(false);
// 设置请求方式（GET/POST）  
        httpUrlConn.setRequestMethod(requestMethod);

        if ("GET".equalsIgnoreCase(requestMethod)) {
            httpUrlConn.connect();
        }

// 当有数据需要提交时  
        if (null != outputStr) {
            OutputStream outputStream = httpUrlConn.getOutputStream();
// 注意编码格式，防止中文乱码  
            outputStream.write(outputStr.getBytes("UTF-8"));
            outputStream.close();
        }

// 将返回的输入流转换成字符串  
        InputStream inputStream = httpUrlConn.getInputStream();
        InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "utf-8");
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

        String str = null;
        while ((str = bufferedReader.readLine()) != null) {
            buffer.append(str);
        }
        bufferedReader.close();
        inputStreamReader.close();
// 释放资源  
        inputStream.close();
        inputStream = null;
        httpUrlConn.disconnect();
        jsonObject = JSONObject.fromObject(buffer.toString());
    } catch (ConnectException ce) {
        log.error("Weixin server connection timed out.");
    } catch (Exception e) {
        log.error("https request error:{}", e);
    }
    return jsonObject;
}

```

这个就是通过https协议向某一个URL通过post或者get提交一个JSON字符串。另外在编程中还有一个常见的向某一个URL请求数据，代码如下：

```

public static String httpRequest(String requestUrl) {
    StringBuffer buffer = null;

    try {
// 建立连接  
        URL url = new URL(requestUrl);
        HttpURLConnection httpUrlConn = (HttpURLConnection) url.openConnection();
        httpUrlConn.setDoInput(true);
        httpUrlConn.setRequestMethod("GET");

// 获取输入流  
        InputStream inputStream = httpUrlConn.getInputStream();
        InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "utf-8");
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

// 读取返回结果  
        buffer = new StringBuffer();
        String str = null;
        while ((str = bufferedReader.readLine()) != null) {
            buffer.append(str);
        }

// 释放资源  
        bufferedReader.close();
        inputStreamReader.close();
        inputStream.close();
        httpUrlConn.disconnect();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return buffer.toString();
}

```

&nbsp;