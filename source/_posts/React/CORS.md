---
title: No 'Access-Control-Allow-Origin' header is present on the requested resource.
tags: 
  - CORS
  - nginx
categories: 
  - Technology
  - 后端开发
---

> No 'Access-Control-Allow-Origin' header is present on the requested resource. 

## CORS是什么？ 
**CORS** （Cross-Origin Resource Sharing，跨域资源共享）是一个系统，它由一系列传输的[HTTP头](https://developer.mozilla.org/zh-CN/docs/Glossary/HTTP_header)组成，这些HTTP头决定浏览器是否阻止前端 JavaScript 代码获取跨域请求的响应。
[同源安全策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy) （**same-origin policy**）默认阻止“跨域”获取资源。但是 CORS 给了web服务器这样的权限，即服务器可以选择，允许跨域请求访问到它们的资源。

## 错误信息 blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
如果请求发送方和服务提供方不在同一个源/域内，服务器返回的响应不包含```Access-Control-Allow-Origin```字段，或者```Access-Control-Allow-Origin```字段和http请求的```origin```不匹配，相应会被浏览器拦截，例如
> ...has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

## 解决方法
### 服务器端 - nginx设置CORS
在nginx对应的site配置文件中打开CORS，同时支持预检请求（preflight request）。

```
#
# Wide-open CORS config for nginx
#
location / {
     if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        #
        # Custom headers and headers various browsers *should* be OK with but aren't
        #
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        #
        # Tell client that this pre-flight info is valid for 20 days
        #
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
        return 204;
     }
     if ($request_method = 'POST') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
     }
     if ($request_method = 'GET') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
     }
}
```

其它服务器打开CORS，请参考[这里]([enable cross-origin resource sharing (enable-cors.org)](https://enable-cors.org/server.html)). 

## 参考
- https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS
- [enable cross-origin resource sharing](https://enable-cors.org/server_nginx.html)
