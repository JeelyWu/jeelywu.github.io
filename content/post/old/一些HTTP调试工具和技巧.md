---
title: 一些HTTP调试工具和技巧
date: 2019-01-10 16:08:02
slug: http-debug
tags:
- http
categories:
- 前端
- 后端
---
今天有些时间，总结一下平常HTTP调试的一些思路、工具和技巧。不够系统，经验之谈。

### 浏览器网络调试工具
一般来说只要不是什么疑难杂症，都可以通过浏览器的网络调试工具调出来。观察状态码/请求头/响应头/响应内容等这些信息，大部分问题就能找到。这也是比较基础的一步，网上教程铺天盖地，不再细说。

### Curl
在很多时候，发出去的HTTP请求怎么也得不到预期的结果。比如刚才还好好的，怎么现在就不行了？又或者明明我这里可以，你那里就是不行呢？对于这些问题，如果浏览器网络调试不能直观得到解决，我通常使用curl来调试。

首先分别将正常的请求，和异常的请求，都复制成Curl，如下图：

![copy-as-cur](/images/2019-01-10-copy-as-curl.png)


复制出来的内容是这样的，也就是一串curl代码：
```
curl 'https://www.baidu.com/' -H 'Connection: keep-alive' -H 'Pragma: no-cache' -H 'Cache-Control: no-cache' -H 'Upgrade-Insecure-Requests: 1' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8' -H 'Accept-Encoding: gzip, deflate, br' -H 'Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7,fr;q=0.6,la;q=0.5,co;q=0.4,de;q=0.3,nb;q=0.2,und;q=0.1,pt;q=0.1,hu;q=0.1,mt;q=0.1,pl;q=0.1'--compressed
```

这就拥有两段curl代码，有时仅仅通过对比这两段代码的差异，就能看出来问题了。

但有时候差异较多，虽然能看出来差异，却并不能确定是哪些差异导致了这些问题。这就是为什么要通过curl来调试了，因为可以通过排除的方式，对比两串curl代码，尝试删掉一些请求头，再拿到终端里执行观察结果，只要耐心，最后一定找到原因 —— 这是个需要耐心的活 :)

根据我的经验，这些问题可能性较大：
- Cookie不同（是否有登录或者授权或者什么别的验证）
- User-Agent不同（服务端对某些浏览器限制，或者禁止爬虫）
- 请求URL不同（还真碰到过，地址写错了！）；
- 传递参数不同;
- Content-Type不同，提交数据时，不同的 Content-Type 决定服务器端会怎么处理数据。

#### curl tips:
curl 可加参数 -i,列出响应头，有时候非常有用，特别是在服务器返回一个重定向时，因为没有响应内容，如果看不到响应头的话，会觉得很纳闷！
![curl-i](/images/2019-01-10-curl-i.png)


### postman

![postman](/images/2019-01-10-postman.png)

Postman也是很多人知道的，用来调试接口非常方便。需要调用接口时，先用postman调试一下，调通了再付诸于代码，会省很多力气。

### Har文件

前两天从Google Chrome的博客中发现Chrome有一个功能，可将请求导出到一个文件，用于保留当时的请求现场。又可将此文件导入到控制台，以便随时都可以分析。
![copy-as-ha](/images/2019-01-10-copy-as-har.png)

这是个非常有用的功能。比如同事之间一起调试，拿到对方的这个har文件，就能知道当时发生了什么；又比如一些不太好复现的场景，保存下来慢慢分析。但这个功能似乎没看到有人介绍过，估计有很多人不知道。


-------
就这些吧。