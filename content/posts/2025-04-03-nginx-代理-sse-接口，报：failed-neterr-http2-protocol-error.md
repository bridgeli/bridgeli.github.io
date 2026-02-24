---
title: nginx 代理 sse 接口，报：(failed) net::ERR HTTP2 PROTOCOL ERROR
author: Bridge Li
type: post
date: 2025-04-03T07:49:30+00:00

categories:
  - Java
tags:
  - nginx
  - SSE

---
前一段时间曾写了一篇关于[Spring MVC 通过 SSE 实现消息推送][1]的小文章，后来系统上线的时候，遇到了一个小问题，打开浏览器的 network，看到接口报：(failed) net::ERR HTTP2 PROTOCOL ERROR，通常是因为 HTTP/2 协议与 SSE 的某些特性不兼容所导致的。SSE 是基于 HTTP 协议的服务器推送技术，它要求连接保持打开状态以便服务器可以持续发送更新给客户端。我们使用的 nginx version 是：nginx/1.26.1

只需要按如下配置即可解决：

```

server {  
listen 80;  
server_name bridgeli.com;

access_log /var/log/nginx/bridgeli_access.log;  
error_log /var/log/nginx/bridgeli_error.log warn;

location ^~ /admin-api/ {  
proxy_pass http://192.168.124.34:8080/;

\# 确保使用HTTP/1.1来支持SSE  
proxy_http_version 1.1;

\# 关闭代理连接的“Connection”头，以避免潜在的问题  
proxy_set_header Connection &#8221;;

\# 增加超时设置，确保长时间连接不会被关闭  
proxy_read_timeout 86400s;  
proxy_send_timeout 86400s;

\# 如果需要禁用HTTP/2（可选）  
\# 注意：这个设置是在server块中，而不是location块中  
\# listen 80 http2 off; 对于HTTP/2协议错误特别有用  
}

location / {  
root /project/www/bridgeli/admin/;  
try_files $uri $uri/ /index.html;  
}  
}

```

 [1]: http://www.bridgeli.cn/archives/773