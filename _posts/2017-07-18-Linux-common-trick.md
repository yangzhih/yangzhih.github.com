---
layout: post
title: "Linux系统下一些有趣的小技巧"
description: "Linux系统下一些有趣的小技巧"
category: articles
tags: [linux, 常用小技巧]
comments: false
---

>本文主要讲述在Linux系统里面有一些鲜为人知却又作用很大的小工具，作者若是发现了一些有趣又实用的小工具会持续更新此博文。

### 录屏命令：script,scriptreplay
>很多时候会遇到这种问题，我们通过命令行的方式把部署某个服务的过程形成了文档，朋友在根据文档做相应部署时出现可能会忽略相关的细枝末节的东西导致部署失败，这是录屏命令就相当直观了。
在做要录屏的操作之前执行script -t 2>time -a cmd命令,time文件保存了时间序列数据，cmd文件保存了录屏内容。此时在命令行窗口执行的操作就会记录与文件中了。执行完操作后使用ctrl+d或者exit退出录屏操作。使用scriptreplay time cmd 命令进行录屏回放。

	[root@centos6 ~]# script -t 2>time -a cmd     #录屏，执行此操作后可执行要录取的命令
	Script started, file is cmd
	[root@centos6 ~]# hostname
	centos6
	[root@centos6 ~]# echo "I love Linux."
	I love Linux.
	[root@centos6 ~]# exit    #退出录屏，或者使用ctrl+D  
	exit
	Script done, file is cmd
	[root@centos6 ~]# scriptreplay time cmd   #回放录屏操作，时间文件在前


	