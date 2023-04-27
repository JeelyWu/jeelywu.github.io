---
title: HTTP之：Host字段和虚拟主机
date: 2019-03-19 16:08:52
slug: http-host-and-vhost
tags:
- http
categories:
- 前端
- 后端
---
一转眼两个多月没有写博客了，还是要把写博客的习惯捡起来，今天就写一点简单的，复习一下Host字段。

HTTP请求头中一般有一个`Host`字段，可能很多人对这个字段习以为常，并不觉得有什么作用。

![host-field](/images/2019-03-19-http-host.png)


我们知道，浏览器会查询域名的DNS，从而就能找到域名背后的服务器ip地址，将请求发往这个ip地址，再从这里接收响应，一次简单的请求就算完成了。

但是这里有一个问题，如果这个服务器托管的不只是一个网站，而是成千上万个网站呢？浏览器的请求发过来，服务器怎么知道你要访问的究竟是哪一个网站呢？这就需要通过Host字段来标识了。

大致过程如下：
1,服务器软件（Web Server）通过配置，给各个域名分别配置cgi处理程序或者简单路径（如果是静态文件的话），例如：

```
server{
    listen 80;
    server_name www.webkit.cc;
    root /path/to/your/root/directory/;
    index  index.html;
}

server{
    listen 80;
    server_name another—website.com;
    root /another/root/directory/;
    index  index.html;
}
```
这就是一个简单的静态文件处理示例，通过给Nginx配置`server_name www.webkit.cc`，那么当来自`www.webkit.cc`的请求时，Nginx根据配置就会读取`第一块server配置`，按照其中的配置来处理。而如果来自`another—website.com`,就会按照`第一块server配置`来处理。

2,那如果知道请求是来自哪一个域名呢？这就需要到`Host`指令了，服务器端软件(如Nginx、Apache)通过解析请求头中的`Host`指令，即可找到对应这个域名的配置，就能正确地分发请求。

这就是`Host`的作用和原理了。在HTTP/1.1中规定，请求头中必须包含`Host`字段，这个字段使得虚拟主机托管成为可能，所谓虚拟主机就是一台服务器可托管很多个网站。