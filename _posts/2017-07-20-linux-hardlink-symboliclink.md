---
layout: post
title: "硬链接与软连接"
description: "硬链接与软连接"
category: articles
tags: [linux, 软连接，硬链接]
comments: 
---


Linux系统中存在两种链接文件，硬链接（hard link）和符号链接（symbolic link）。符号链接也称为软连接。想了解清楚这两种链接文件的区别并不容易，首先要清楚Linux文件系统的相关知识。
我们知道文件有文件名和数据。而Linux的文件系统在存储文件时分为两个部分，用户数据（userdata）和元数据（metadata）。用户数据是文件的真实数据存储文件系统的data block中，元数据存储在一个iNode的节点块中，包括文件的iNode号，权限，大小，时间属性（atime,ctime,mtime），所属者/组等属性信息（但并不包括文件名）。
## iNode
对于文件系统来讲，iNode（index node）才是文件的唯一标识，并非文件名，文件名只是符合人类记忆习惯而实现的一种方式。这样就可用下图来说明计算机是如何通过文件名来访问的文件的了。
  
![](http://ot9scj6tc.bkt.clouddn.com/001.jpg)


在Linux操作系统中，可以使用stat或者ls -i命令查看文件的iNode号,使用df  –i 命令查看整个文件系统的iNode节点使用情况。笔者说过，iNode号相对于文件系统来具有唯一性，对于不使用raid或者lvm技术的传统文件系统的来说就是在“分区”内iNode是文件访问的唯一标识。

为了解决文件共享的问题（笔者自认为），Linux在文件系统设计上支持链接文件。分为硬链接和符号链接。
## 硬链接
当iNode号具有多个文件名的时候，称这种文件为硬链接文件，硬链接文件和源文件在系统下具有相同的“地位”。为了更直观的理解，我引用了鸟哥私房菜书中的插图。

![](http://ot9scj6tc.bkt.clouddn.com/hardlink.jpg)

由于硬链接的元数据与源文件的完全一致（就是同一条元数据），所以有了它的一系列特性。

1，	不能跨文件系统；

2，	有相同的iNode和data block； 

3，	硬链接只能对已经存在的文件进行创建；

4，	不能针对目录文件进行创建，Linux文件系统在设计时对目录默认创建了两个硬链接， .  ..  如果再支持目录创建硬链接的话会生成目录环可能会造成死锁（笔者一家之言）；

5，	删除文件时，必须删除所有的硬链接才能真正释放data block.


## 软连接

与硬链接不同，软连接是一个独立的文件，与源文件的元数据和用户数据都不同。软连接的的数据块中存储的只是指向源文件的路径。当然这里也这里也引用鸟哥图来让读者直观的理解。

![](http://ot9scj6tc.bkt.clouddn.com/symboliclink.jpg)

相比于硬链接来说，我认为软连接打破了他的诸多限制。

1，	软连接有自己的元数据，所以可以对其进行权限控制（作用于原文件）

2，	可跨越文件系统（包括网络）创建软连接

3，	可对不存在的文件和目录创建（打破不能链接目录的限制）

4，	创建软连接，源文件链接数不增加

5，	删除软连接时不影响源文件


## 小实验

>验证文件访问是通过iNode节点而非目录。验证方法很简单，对文件进行移动，重命名后查看iNode节点号是否变化。当然还有一点很重要，对于Linux来讲目录也是文件，所以在同一个目录下不能建立同名的文件和目录。

	[yzh@centos6 workspace]$ touch test
	[yzh@centos6 workspace]$ ls -i test     #查看test文件的iNode号为1704277
	1704277 test
	[yzh@centos6 workspace]$ mkdir dir1
	[yzh@centos6 workspace]$ mv test dir1/    
	[yzh@centos6 workspace]$ ls -i dir1/test #移动test文件到不同的目录下，iNode节点不变
	1704277 dir1/test
	[yzh@centos6 workspace]$ mv dir1/test test.txt    #对test文件进行重命名，iNode节点号依旧不变，反映出iNode号是文件访问的唯一标识
	[yzh@centos6 workspace]$ ls -i test.txt 
	1704277 test.txt
	[yzh@centos6 workspace]$ ls
	dir1  test.txt
	[yzh@centos6 workspace]$ mkdir test.txt 
	mkdir: cannot create directory `test.txt': File exists
	

>验证硬链接的相关特性
>ln source hardlink 命令建立硬链接

	[root@centos6 app]# df -T     #查看文件系统信息，笔者在/app分区下做实验
	Filesystem     Type  1K-blocks    Used Available Use% Mounted on
	/dev/sda2      ext4   51475068 4751996  44101632  10% /
	tmpfs          tmpfs   1019172      76   1019096   1% /dev/shm
	/dev/sda3      ext4   40185208   49016  38088192   1% /app
	/dev/sda1      ext4     999320   40008    906884   5% /boot
	[root@centos6 app]# touch test;ll -i test   #创建并查看test文件的iNode号，以及第三字段的链接数
	11 -rw-r--r--. 1 root root 14 Jul 20 13:17 test
	[root@centos6 app]# ln test /boot/test_hardlink  #在boot分区创建硬链接，提示错误，创建硬链接不能跨越文件系统
	ln: creating hard link `/boot/test_hardlink' => `test': Invalid cross-device link
	[root@centos6 app]# ln test test_hardlink   #创建硬链接成功
	[root@centos6 app]# ll -i	# 查看源文件和链接文件的属性信息，完全一致，链接数增加1，硬链接文件显示是普通文件，与源文件具有相同的'地位'
	total 0
	11 -rw-r--r--. 2 root root 0 Jul 20 13:15 test
	11 -rw-r--r--. 2 root root 0 Jul 20 13:15 test_hardlink
	[root@centos6 app]# echo "I love linux." >test_hardlink 
	[root@centos6 app]# cat test   #对硬链接文件进行修改，源文件同步修改，
	I love linux.
	[root@centos6 app]# ln test1 test1_hardlink  #不能对未存在的文件创建链接
	ln: accessing `test1': No such file or directory
	[root@centos6 app]# ln /app app_link   #不能对目录进行创建链接
	ln: `/app': hard link not allowed for directory
	[root@centos6 app]# rm -f test  #删除源文件后，数据依旧存在，只有把所有的硬链接文件都删除掉，磁盘空间才会被释放
	[root@centos6 app]# cat test_hardlink 
	I love linux.


>验证软连接特性
>ln -s source  symlink 命令建立软连接


	[root@centos6 app]# touch test
	[root@centos6 app]# ll -i     #关注iNode号与链接数
	total 0
	11 -rw-r--r--. 1 root root 0 Jul 20 13:31 test
	[root@centos6 app]# ln -s test test_symlink  #创建软连接 
	[root@centos6 app]# ll -i  #软连接和源文件有不同的元数据，不是同一个文件
	total 0
	11 -rw-r--r--. 1 root root 0 Jul 20 13:31 test
	12 lrwxrwxrwx. 1 root root 4 Jul 20 13:31 test_symlink -> test
	[root@centos6 app]# chmod 600 test_symlink   #对软连接进行权限更改后，效果作用于源文件
	[root@centos6 app]# ll -i
	total 0
	11 -rw-------. 1 root root 0 Jul 20 13:31 test
	12 lrwxrwxrwx. 1 root root 4 Jul 20 13:31 test_symlink -> test
	[root@centos6 app]# ln -s /app/test /boot/test_symlink  #可以跨越文件系统进行创建软连接
	[root@centos6 app]# ll -i  /boot/test_symlink
	23 lrwxrwxrwx. 1 root root 9 Jul 20 13:38 /boot/test_symlink -> /app/test
	[root@centos6 app]# echo "I love linux." >test_symlink #对软连接的写操作也是写入到原文件
	[root@centos6 app]# cat test
	I love linux.
	[root@centos6 app]# rm -f test_symlink  #删除软连接后不对源文件有影响 
	[root@centos6 app]# cat test 
	I love linux.

## 软连接创建过程中遇到的那些坑

在软连接的创建过程牵扯到一个使用相对路径和绝对路径的问题，在使用相对路径链接到源文件时的参考点是以软连接所在的目录为基准，而非当前工作目录，所以，在使用相对路径创建软连接时，一定要尤为注意。这里笔者有一个小经验， 对于ln -s source  symlink     这个命令来说，就是为了创建软连接，所以可以在敲命令时先敲入软连接的路径 ，这就无所谓相对路径和绝对路径了（建议使用绝对路径，可以以链接文件所在路径推出原文件相对于链接文件的路径）。
读者可能会想既然使用相对路径会出现这么多的麻烦，为什么还有人使用呢？因为有时需要对文件以及他的软连接进行'同步'迁移（相对路径不变），这时软连接还是能够连接到原文件，使用绝对路径就没有这种方便。我用下面代码展示一下。

	[root@centos6 app]# touch test
	[root@centos6 app]# mkdir dir
	[root@centos6 app]# cd dir/
	[root@centos6 dir]# ln -s ../test test_link1   #用相对路径进行创建软连接
	[root@centos6 dir]# ln -s /app/test test_link2 #用绝对路径进行创建软连接
	[root@centos6 dir]# tree /app/
	/app/
	├── dir
	│   ├── test_link1 -> ../test
	│   └── test_link2 -> /app/test
	└── test
	1 directory, 3 files

![](http://ot9scj6tc.bkt.clouddn.com/adaga.png)

如上图所示，使用绝对路径在迁移时软连接会失效。因此，使用绝对路径和相对路径建立软连接需要根据具体环境而言，而有优缺点。