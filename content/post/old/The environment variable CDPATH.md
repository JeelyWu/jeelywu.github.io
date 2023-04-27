---
title: The environment variable CDPATH
date: 2021-06-18 23:14:55
slug: CDPATH
tags:
- shell
---
As all *unix system users know, we have a set of environment variables. The most famous is 'PATH' which contains a series of directories. When we type a command in the shell, the shell will find each of the directories to find out if there is an executable program named the command we typed.

So, what is CDPATH?

We all know, `cd` is a basic command to navigate between directories. When we type `cd some-directory`, it means that if the current directory has a subdirectory called `some-directory`,  we want entry it.

The point here is, cd will search subdirectories based on the current directory.  But in daily work, We mainly work in several directories. For me,      I frequently jump into ~, ~/code, ~/onedrive. Is there any way to jump to subdirectories on ~ or ~/code without navigating to ~ or ~/code first?

This is the role of CDPATH. Like PATH, it contains a series of directories.
By default, the variable is empty and represented the current directory. You can config it in your shell config file like this: `CDPATH=~:~/code:~/onedrive`

Then if you type `cd some-directory` again in your shell, the cd will search each of the directories configured in the CDPATH, to see if there is a subdirectory named `some-directory` and jump into it.

So after configuring the mainly work directories to CDPATH could speed up your movement between folders.

This is the explanation of CDPATH.

---

By the way, this variable is useful when you don't want to use the third-party navigate utils, such as autojump, z( a plugin of zsh). In my opinion, if you use third-party navigate utils, it is much more convenient than simply config CDPATH.