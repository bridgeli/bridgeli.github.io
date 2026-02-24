---
title: 'javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target'
author: Bridge Li
type: post
date: 2024-04-21T05:46:06+00:00

categories:
  - Java
tags:
  - HTTPS
---
自从 ChatGPT 横空出世之后，突然没了写博客的兴趣，因为无论有什么问题，随便往 ChatGPT 一帖，结果就哗哗的出来了，今天这个问题就是。前一段时间和某公司联调接口，对方的接口测试环境好好的，一上线调不通，报错信息就是标题，只见贴到 ChatGPT，ChatGPT 回答如下：

这个错误通常是由于缺少信任证书或证书链不完整导致的。解决方法通常有两种：

1. \*\*添加证书到信任库\*\*：获取正确的证书，并将其添加到 Java 的信任库中。可以使用 \`keytool\` 工具来完成此操作。具体步骤如下：

&#8211; 首先，使用以下命令导出证书：  
```  
keytool -printcert -rfc -sslserver <hostname> -port <port>  
```  
其中 ```<hostname>``` 和 ```<port>``` 是你要访问的 HTTPS 接口的主机名和端口号。

&#8211; 然后将证书导出到一个文件中。

&#8211; 最后，使用以下命令将证书添加到 Java 的信任库中：  
```  
keytool -importcert -file <certificate-file> -keystore <path-to-truststore> -alias <alias>  
```  
其中 ```<certificate-file>``` 是你导出的证书文件，```<path-to-truststore>``` 是 Java 信任库的路径，```<alias>``` 是证书的别名。

2. \*\*忽略证书验证\*\*：如果你确定你要连接的服务器是安全的，并且不需要严格的证书验证，你可以在 Java 代码中忽略证书验证。但这不是推荐的做法，因为可能会导致安全风险。你可以使用 \`TrustManager\` 接口实现自定义的信任管理器，然后将其应用于 SSL 连接。下面是一个简单的示例代码：

```  
import javax.net.ssl.*;  
import java.security.cert.X509Certificate;

public class SSLCertificateIgnore {  
public static void main(String[] args) throws Exception {  
// Create a trust manager that does not validate certificate chains  
TrustManager[] trustAllCerts = new TrustManager[]{  
new X509TrustManager() {  
public X509Certificate[] getAcceptedIssuers() {  
return null;  
}  
public void checkClientTrusted(X509Certificate[] certs, String authType) {  
}  
public void checkServerTrusted(X509Certificate[] certs, String authType) {  
}  
}  
};

// Install the all-trusting trust manager  
SSLContext sc = SSLContext.getInstance("SSL");  
sc.init(null, trustAllCerts, null);  
HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());

// Now you can access the HTTPS endpoint without certificate validation  
}  
}  
```

请注意，这种方法会降低安全性，因为它会使你的应用程序容易受到中间人攻击。因此，最好的做法是第一种方法，即添加正确的证书到信任库中。

因为调用三方接口，不可能使用第一种方法，所以直接采用第二种，把代码贴进去，解决，完美，一点脑子都不用动！