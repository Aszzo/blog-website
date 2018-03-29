---
layout: '[layout]'
title: Nginx开启和配置gzip压缩
date: 2017-12-5 14:38:05
comments: true
toc: true
tags:
	- 前端
	- nginx
---
<strong style='font-size:18px'>前言</strong>

nginx是一个高性能的web服务器，在实际应用中具有十分重要的意义，合理配置nginx可以有效提高网站的相应速度。

本文介绍如何开启和配置nginx的gzip功能。

Nginx的压缩输出有一组gzip压缩指令来实现。

相关指令位于`http{…}`两个大括号之间。
<!-- more -->
<strong style='font-size:18px;color:#428BD1'>开启gzip</strong>
```js
# 开启gzip
gzip on;

# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
gzip_min_length 1k;

# gzip 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间
gzip_comp_level 6;

# 进行压缩的文件类型。javascript有多种形式。
# 其中的值可以在 mime.types 文件中找到。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;

# 是否在http header中添加Vary: Accept-Encoding，建议开启
gzip_vary on;

# 禁用IE 6 gzip
gzip_disable "MSIE [1-6]\.";
```
关于具体的参数说明可以参考 <a href='http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html'>nginx 的文档</a>。

<strong style='font-size:18px;color:#428BD1'>开启缓存</strong>
```js
location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ {
    access_log off;
    expires 30d;
}

location ~* ^.+\.(css|js|txt|xml|swf|wav)$ {
    access_log off;
    expires 24h;
}

location ~* ^.+\.(html|htm)$ {
     expires 1h;
}
```

参考资料：<a href='https://www.darrenfang.com/2015/01/setting-up-http-cache-and-gzip-with-nginx/'>加速nginx: 开启gzip和缓存</a>。