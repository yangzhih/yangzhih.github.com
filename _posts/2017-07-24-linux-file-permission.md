---
layout: post
title: "Linux文件标准权限"
description: "Linux文件管理"
category: articles
tags: [linux, 权限]
comments: false
---

## Linux文件属性

在弄清楚Linux文件权限之前，首先要知道Linux的文件属性都有哪些，对于Linux文件的属性也就是‘ls -l’命令显示的长格式显示。

![](http://ot9scj6tc.bkt.clouddn.com/ls-l.png)

如上图所示，第一字段表示文件的类型和权限，第二字段表示链接数（硬链接的数量），第三字段为所有者，第四字段为所属组，第五字段表示文件大小，第六字段表示文件的创建或者最近的修改时间，最后表示文件名。<br/>
在Linux系统中，文件类型有7种，分别为目录文件（d），普通文件(-),链接文件（l）,块设备文件（b）,字符设备文件（c）,管道文件（p）和socket文件（s）。在第一字段的后九个字段表示文件的权限。Linux文件系统把对文件的访问权限分为三个类型：所有者（u），所属组(g)，其他人(o)，而这九个字段每三个一组，分别表示这三种人对该文件的权限。九个字段分为3组，每组均为‘rwx’三个参数的组合，读（r）,写（w）,执行（x）。
在文件属性的操作主要是对文件的所有者和所属组的修改，常用命令有chown和chgrp。以test为例：

	[root@centos6 app]# ll test 
	-rw-r---w-. 1 root root 0 Jul 22 16:36 test
	[root@centos6 app]# chown yzh test    #修改文件所有者为yzh
	[root@centos6 app]# ll test 
	-rw-r---w-. 1 yzh root 0 Jul 22 16:36 test
	[root@centos6 app]# chgrp yzh test   #修改文件所属组为yzh
	[root@centos6 app]# ll test 
	-rw-r---w-. 1 yzh yzh 0 Jul 22 16:36 test
	[root@centos6 app]# chown zachary.zachary test   #chown不仅可以修改所有者，还可以修改文件所属组，中间以.或:分隔
	[root@centos6 app]# ll test 
	-rw-r---w-. 1 zachary zachary 0 Jul 22 16:36 test
	
	[root@centos6 app]# ll dir/test1 dir/ -d
	drwxr-xr--. 2 root root 4096 Jul 23 03:15 dir/
	-rw-r--r--. 1 root root    0 Jul 23 03:15 dir/test1
	[root@centos6 app]# chown -R yzh.yzh dir     #递归更改文件属性，chgrp命令也支持该选项
	[root@centos6 app]# ll dir/test1 dir/ -d
	drwxr-xr--. 2 yzh yzh 4096 Jul 23 03:15 dir/
	-rw-r--r--. 1 yzh yzh    0 Jul 23 03:15 dir/test1

##  Linux标准权限

上文简单介绍了属性中代表权限的标识，下面简单介绍一下文件权限的更改以及权限的作用。<br/>
首先，文件权限是文件系统支持的一种访问规则，不同的系统有着不同的规则，这里所讲述的文件系统是基于Linux系统的（ext或者xfs）,对于fat或者ntfs系统挂载到Linux主机后，并不适用。<br/>

### 权限更改
	
Linux权限更改使用chmod命令，而权限更改支持两种更改方式，即文字法和数字法。

- 文字法

文字法更改权限规则是 chmod [ugoa][+-=][rwx] filename  这种方式。其中ugoa四个字符分别代表文件的访问者（所有者，所属组，其他人，所有人），以对test文件的权限更改为例。

	[root@centos7 app]# ll test 
	-rw-r--r--. 1 root root 5427200 Jul 25 08:21 test
	[root@centos7 app]# chmod u-r,g+w test    #同时针对两种人更改，以逗号分隔
	[root@centos7 app]# ll test 
	--w-rw-r--. 1 root root 5427200 Jul 25 08:21 test
	[root@centos7 app]# chmod a+x test    #给予所有人可执行权限
	[root@centos7 app]# ll test
	--wxrwxr-x. 1 root root 5427200 Jul 25 08:21 test
	[root@centos7 app]# chmod u=rw,g=r,o=- test #使用=号直接赋予文件权限，没有权限者使用-
	[root@centos7 app]# ll test 
	-rw-r-----. 1 root root 5427200 Jul 25 08:21 test

- 数字法

数字法更改权限，文件属性第一字段的后九个字符分为三组，每组表示一种访问者，把每组的rwx值看为1，‘-’看为0，即r(4)w(2)x(1),组成一个8进制数表示该访问者的权限，比如rwx表示7，rw-(6)..-w-(2)---(0)。三组访问者组合起来之后就是一个三位的8进制数表示文件的标准权限。‘rw-r--r--’使用数字法表示为644，即所有者对其有读写权限，所有组和其他人对其有读的权限。下面依旧以test为例来讲述该方法：

	[root@centos7 app]# chmod 644 test  #更改所有者有读写权限，所属组和其他人为读权限
	[root@centos7 app]# ll test 
	-rw-r--r--. 1 root root 5427200 Jul 25 08:21 test
	[root@centos7 app]# chmod 755 test #更改所有者有所有权限，所属组合其他人有读和执行的权限
	[root@centos7 app]# ll test 
	-rwxr-xr-x. 1 root root 5427200 Jul 25 08:21 test

### 目录文件的读写执行权限

读写执行对于目录文件和对于普通文件不太一样，目录的读权限表示可以查看该目录下的文件列表，执行表示可以进入该目录下，写权限表示可以对其文件内容（可理解文件列表）可以操作，也就是可以在该目录下进行新建或者删除文件，但是需要x(执行)权限的配合。
	
	[root@centos7 app]# mkdir dir
	[root@centos7 app]# ll dir -d
	drwxr-xr-x. 2 root root 6 Jul 25 09:04 dir
	[root@centos7 app]# chmod o=wx dir  #赋予其他人写和执行权限，无读权限
	[root@centos7 app]# su yzh
	[yzh@centos7 app]$ touch dir/test  #在dir目录下新建文件成功
	[yzh@centos7 app]$ ll dir/test
	-rw-rw-r--. 1 yzh yzh 0 Jul 25 09:06 dir/test  #可以查看文件的属性信息
	[yzh@centos7 app]$ ls dir
	ls: cannot open directory dir: Permission denied
	[yzh@centos7 app]$ ll dir -d
	drwxr-x-wx. 2 root root 18 Jul 25 09:06 dir

上面显示在yzh对dir目录无读权限的情况下可查看test文件属性的信息。在对某个目录有执行权限后，若事先知道该目录下有某个文件并对其有读权限，用户即可读出此文件。可以参阅查阅笔者之前博客或者搜索文件系统的inode相关知识。

### umask

在创建文件和目录的时候我们会发现root用户创建的文件默认为644权限，目录为755权限。这就归根于umask的作用，在Linux文件系统设计之初出于安全性考虑文件试图创建为666权限，目录创建为777权限，但是否创建为666或者777权限要根据umask的值，umask表示文件系统不允许创建的权限值。以022为例就是不允许所属组和其他人有写权限，所以创建的文件为644，目录为755权限。<br/>
从专业角度来讲，创建的文件是何权限是用666，777与umask进行异或运算得出来的（xor）。详细创建过程，如下图所示：

![](http://ot9scj6tc.bkt.clouddn.com/mask.png)

	
