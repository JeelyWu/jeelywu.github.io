---
title: 编译安装PHP
date: 2019-05-04 16:10:18
slug: how-to-compile-php
tags:
- php
categories:
- 后端
---

如果要安装PHP，一般来说是直接下载已经编译好的二进制文件，在PHP文档中有各个平台的[安装指导](https://www.php.net/manual/zh/install.php)。但是如果想认真研究一下PHP，自己手动编译安装一下，显然是很有必要的。

### 下载源码
PHP源码可以在GitHub上[找到](https://github.com/php/php-src)，直接克隆下来即可，也可借此机会一窥PHP的源码结构。

```
git clone git@github.com:php/php-src.git
```

默认的master分支似乎是在开发 PHP 8 ，我们可以切换到一个稳定分支，比如：`git checkout PHP-7.1.29`

### 编译
源码下载之后，就可根据文档的指示开始编译PHP了。其实PHP就是一个由C语言编写的程序，如果熟悉Linux环境下一般C语言的编译并没有什么困难。


#### 第一步：buildconf
```
./buildconf
```
这里首先是运行源码下的 ./buildconf 文件，这个文件按照[注释](https://github.com/php/php-src/blob/master/buildconf)应该是一个autoconf的包装器，用来生成编译PHP所需的文件。
> A wrapper around Autoconf that generates files to build PHP on *nix systems.

由于我没写过很多C语言程序，对autoconf并不太了解，这里先不深究。

./buildconf 运行过后，注意观察就会发现，这里目录下生成了 `configure` 文件，用于设置编译PHP的各种选项。


#### 第二步：configure

我们可以通过 `./configure -h` 来查看所有的编译选项。这里的选项非常的多，可以根据自己的需求自行配置。为了不影响到全局环境，我们可以通过`--prefix`选项来指定安装路径。另外也可以通过`--enable-fpm`来给我们生成`php-fpm`:

```
./configure --disable-all --prefix=自定义路径 --enable-fpm
```

运行以上命令，configure 会检查编译所需的各个组件，如果环境不满足会有报错，提示缺少啥啥啥。基本上就是少了啥就安装啥，装好了之后，再继续 configure 即可。

#### 第三部：编译 & 安装
如果上一步 configure 顺利通过了，就能看到目录下生成了 Makefile 文件， 运行

```
make // 编译
make install // 安装到prefix指定位置
```

通过 make && make install 之后PHP就安装好了，这时候进到之前指定的安装目录就能看到

可以看到目录结构是这样：

```
.
├── bin
│   ├── phar -> phar.phar
│   ├── phar.phar
│   ├── php
│   ├── php-cgi
│   ├── php-config
│   ├── phpdbg
│   └── phpize
├── etc
│   ├── php-fpm.conf.default
│   └── php-fpm.d
├── include
│   └── php
├── lib
│   └── php
├── php
│   ├── man
│   └── php
├── sbin
│   └── php-fpm
└── var
    ├── log
    └── run
```

我们重点关注 bin 目录和 sbin 目录，因为这是编译出来的二进制文件。
在bin目录中：

- phar.phar php打包工具，把一堆PHP代码打包成一个phar包。我没有亲自打过包，参考 [维基百科](https://zh.wikipedia.org/wiki/PHAR_(%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F)) 
- php ，命令行的php , 如果你运行 `./bin/php -v` 就能看到版本号和sapi类型:`PHP 7.1.29 (cgi)`
- php-cgi ，cgi模式的php，同样如果运行 `./bin/php-cgi -v` , 能看到版本号和sapi类型`PHP 7.1.29 (cli)`
- php-config 输出编译选项的工具，试试运行 `./bin/php-config` 就能看到刚才编译时的所指定的选项。
- phpdbg 调试用的
- phpize 动态安装扩展用的

sbin目录下：

- php-fpm 

关于PHP如何运行，命令行下和Web下是如何跑起来的，PHP 如何与 Nginx 交互，cgi、fastcgi、php-fpm 待有更深理解再写博文。
