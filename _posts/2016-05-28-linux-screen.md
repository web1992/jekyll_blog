---
layout: post
title:  "linux screen"
date:  2016-05-28 17:50:00 +0800
categories: linux
tags: linux
keywords: linux,web1992
---

linux screen的使用

<!--more-->

--------

- [screen使用技巧](http://www.cnblogs.com/mchina/archive/2013/01/30/2880680.html "使用技巧")
- [screen如何写日志](http://www.linuxidc.com/Linux/2014-09/106217.htm "使用技巧")

sh 中使用screen

```sh
	
	#!/bin/sh
	
	echo "run"

	screen -dm  test &&  echo "test" >a.txt

	# screen -r test 可以链接到此screen了
```