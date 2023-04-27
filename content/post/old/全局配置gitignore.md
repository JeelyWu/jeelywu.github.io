---
title: 全局配置gitignore
date: 2019-03-23 16:09:36
slug: git-global-ignore
tags:
- git
categories:
- 工具
---
今天才知道 Git 可以设置全局的 gitignore, 通过全局设置gitignore ,可以省去很多重复配置。比如：

- 编辑器生成的配置文件 .idea/ .vscode
- 系统生成的无关文件 .DS_Store
- vim缓存文件：xxx.swp
- 各种日志文件

等等。完全可以设置在全局 gitignore 中。

首先确认一下是否已有全局 gitignore: 
`git config --get core.excludesfile` 这个命令查询是否配置有全局gitignore，如果已配置有，则能看到文件路径

如果没有，则可以创建：`git config --global core.excludesfile '~/.gitignore'`。这个文件路径和文件名是可以自定义的。这一步完成后，则可以通过上一步查看配置看到这个`~/.gitignore` 了。或者也可以直接查看 .gitconfig 文件，也能看到增加了一个配置项：`excludesfile = ~/.gitignore`

这时候编辑这个全局的`.gitignore`文件就行了。