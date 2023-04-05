---
title: "Laravel Tips: Is the code running in http request? "
date: 2023-04-01T15:57:10+08:00
categories: 
- laravel
tags:
- laravel
---

We know, the code in Laravel can run in different environment, such as http request, queue, artisan command and even in Tinker.

Sometimes, we need to know the if the code is running in a http request or not. 

Fortunately, Laravel provide a function to do this: `app()->runningInConsole()`. 

If the code is running in a http request, it will return `false`, otherwise, such as in queue, artisan command, or in Tinker it will return `true`.

But, unfortunately,if you want to know the exact environment when it returns true, it seems not a convenient way to do that.