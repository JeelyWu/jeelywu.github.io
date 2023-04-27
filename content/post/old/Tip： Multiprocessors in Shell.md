---
title: Tip： Multiprocessors in Shell
date: 2021-07-12 15:57:36
slug: multiprocess-in-shell
tags:
- English
---

Last week, I need to compress more than 20k png pictures in Shell. By use the `optipng`, it's easy to do. There command in Shell like this:
```bash
optipng *.png --dist ./dist
```

But I need to deal with more than 20k pictures, if I use the command above to compress pictures one by one, it's too slow!

So I think, Is there anyway to use multiprocess to speed up?

The shell language seems not to provide APIs to deal with multiprocess programming.

After searching google, I find two ways to do this.
- Open more than one Shell window, each one does the same things.
- Using script language like python, create multiprocess then call the shell command in each process.

We know, in multiprocess programming or multi-threads programming, we need to avoid the race condition.

Such as, in this case, we need to avoid the `optipng` process not to deal with the same picture at the same time(And need to avoid a `optipng` process to deal with a picture which already compressed! ). Since the Shell lacks logic control, I think the second way is more appropriate.
