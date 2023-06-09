---
title: 为什么说PHP不支持Unicode编码
date: 2018-09-29 16:04:35
slug: php-and-unicode
tags:
- php
categories:
- 后端
---

经常看到有说法：PHP不支持Unicode，或者说PHP在底层不支持Unicode。虽然我知道PHP编码很蛋疼，各种字符串处理函数非常不规范，但也还能显示中文，一直没弄明白这个不支持Unicode是什么意思。花了一些时间来梳理这方面的信息。

先从一个例子来引入：
一个PHP脚本如下，假设文件的编码是UTF-8:

```php
//文件编码UTF-8
echo strlen("中文"); // 6
echo substr("中文",0,1) // 乱码
echo substr("中文",0,3) // 中
```
很奇怪吧，从上面看，似乎把一个汉字当成了3个字符。这就要从PHP对于字符串的存储上说起了。

我总结了一下，如下：

- PHP的字符串，是由字节(byte)组成的数组构成的。也就是说，类似于C语言 `char a[3] = "abc"` 这样，一个字符占据一个字节。 
- 除此之外，并没有存储文本的编码信息，也就是说PHP并不知道这些字符串的二进制数据，应该对应怎样的编码。
- 再进一步，PHP会按照脚本文件的编码，来决定字符串的编码。就比如：`$string = "中文";`，如果脚本文件是UTF-8，就会把`中文`的UTF-8的编码：`E4B8ADE69687`给保存起来。
- 再进一步，如前说所，PHP并不保存字符串的编码信息。所以即便`中文`保存为:`E4B8ADE69687`,在字符串原生函数看来，都只是一串二进制数。所以，PHP原生字符串函数只能操作单字节字符！就是把一个字节当做一个字符来处理！

如果想明白了上面几点，上面的代码例子就自然明白了：
```php
//文件编码UTF-8
echo bin2hex("中文"); // 可以看到，"中文"对应的二进制就是：e4b8ade69687
echo strlen("中文"); // 所以按照单字节来统计长度，就是6 
echo substr("中文",0,1) // 取0到1个字节，也就是e4，并不对应某个字符的编码，所以乱码
echo substr("中文",0,3) // 取0到3个字节，刚好把`中`的编码取出来
```
同理，如果把文件编码换成GBK或者别的，再实验也会得到类似的结果，只不过GBK一个汉字占2字节。

那么到现在，基本可以明白了`PHP底层不支持unicode`到底说的是什么了，总结如下：

> PHP字符串不保存字符的编码信息，所以原生操作函数，并不知道二进制数据该如何对应文本，只能【假设】一个字符对应单个字节。这样在处理英文等ascii码时够用了，但对于中文等【多字节字符】，就会出错了。

而作为反面，我们可以看看所谓底层支持Unicode的语言的情况：
```js
var string = "中文"
console.log(string.length); // 2
string.substr(0,1) // 中
```

可以看到，在JS中，能正确识别和处理多字节字符。也就是在存储时，把文本的编码信息也一并存储。（这里我猜测是保存的是文本的Unicode值，并不太确定，因为不了解JS的底层原理）

那么这里就有疑问了，PHP中如何才能正确处理多字节字符呢？答案就是`mbstring`扩展（具体可看：http://php.net/manual/zh/book.mbstring.php）。所谓mbstring,也就是：multi-byte string ,多字节字符串。

这套扩展中，有一系列与原生字符串函数对应的函数，能用来正确处理多字节字符的情况。如：`strlen` 对应 `mb_strlen` …… 这些对应函数中，基本和原生函数一致，只不过通常多了一个可选参数：编码。

举例如下：

```php
// 脚本类型为UTF-8
echo strlen("中文"); // 6
echo mb_strlen("中文","UTF-8"); //2  使用mb_strlen ，并传入编码 utf-8, 就会把二进制E4B8ADE69687当做utf-8的处理能正确处理
echo mb_strlen("中文"); //2  如果不传编码UTF-8,则函数会自动确定编码，文档说：如果省略，则使用内部字符编码。所以这里也当做UTF-8来处理。
echo mb_strlen("中文","GBK"); //3，如果传入编码GBK，则：e4b8ade69687会被当做gbk来处理，一个gbk字符占2字节，所以为：3
```
