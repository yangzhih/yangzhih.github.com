---
layout: post
title: "Linux文件管理--ACL访问控制列表"
description: "Linux文件管理"
category: articles
tags: [linux, 权限]
comments: false
---
## ACL访问控制列表

ACL（Access Contrl List）访问控制列表的目的就是为了提供Linux传统权限设置之外的针对特定文件或者特定访问者的权限设定。

- 针对特定用户设置权限
- 针对特定组设置权限
- 针对特定目录设定权限，在该目录下新建的文件自动拥有指定的acl默认设置
- ACL生效 顺序：所有者，自定义用户，自定义组，所属组，其他


ACL也是基于文件系统支持的一种文件访问规则，本文所用环境是ext（xfs）文件系统，查看你的文件系统是否支持acl可以执行下列命令

	[root@centos6 ~]# dumpe2fs /dev/sda1|grep "Default mount options"

在CentOS7中的文件系统默认支持ACL，而在CentOS6-版本中，只有在安装系统时建立的文件系统才会默认支持acl，安装系统后新添加的Linux文件系统默认不支持ACL,读者可以在新建文件系统后执行下列命令让其支持ACL访问控制列表。

	[root@centos6 ~]# tune2fs –o acl /dev/sdb1
	[root@centos6 ~]# mount –o acl /dev/sdb1 /mnt/test

## 设置ACL

关于设置ACL的命令很简单，一个setfacl命令用来设置ACL，getfacl用来取得ACL

	setfacl [-bkRd] [{-m|-x} acl参数]  filename
	
	-b:清空所有ACL记录
	-k:删除默认的ACL设置
	-R：递归设置ACL，针对目录
	-d:设置默认ACL，只针对目录，在该目录下新建的文件会拥有该默认ACL设置
	-m:添加或更改指定的ACL条目，不能与-x同时使用
	-x:删除指定的ACL条目
	
### 给特定用户添加ACL

以实验为例：

	[root@centos6 app]# ll test
	-rw-r--r--. 1 root root 0 Jul 23 05:47 test
	[root@centos6 app]# setfacl -m u:yzh:rw test 
	[root@centos6 app]# setfacl -m u:zachary:rw test
	[root@centos6 app]# su yzh
	[yzh@centos6 app]$ echo "I love Linux." > test  ## yzh用户对其有了读写权限 
	[yzh@centos6 app]$ cat test 
	I love Linux.
	[yzh@centos6 app]$ getfacl test 
	# file: test
	# owner: root
	# group: root
	user::rw-
	user:yzh:rw-  	 	##新增的ACL条目
	user:zachary:rw-	 ##
	group::r--
	mask::rw-
	other::r--
	[yzh@centos6 app]$ exit
	exit
	[root@centos6 app]# setfacl -x u:yzh test  #删除指定条目时不需加具体权限 
	[root@centos6 app]# getfacl test   #查看删除结果
	# file: test
	# owner: root
	# group: root
	user::rw-
	user:zachary:rw-
	group::r--
	mask::r--
	other::r--
	[root@centos6 app]# setfacl -b test  ##清空ACL
	[root@centos6 app]# getfacl test 
	# file: test
	# owner: root
	# group: root
	user::rw-
	group::r--
	other::r--

### 给特定组添加ACL权限
	
给特定组添加ACL和特定组类似这里不再详述：	
	[root@centos6 app]# setfacl -m g:tom:rwx test   #与给用户添加时条目信息中的u换为g

### 给目录添加默认的ACL权限

给目录添加默认权限可以执行以下命令中的任意一个

	[root@centos6 app]# setfacl -d -m g:tom:r /testdir/dir
	[root@centos6 app]# setfacl -m d:g:tom:r /testdir/dir

在给目录添加权限的时候会有很多小问题，若不注意，也许会无法访问该目录
1，检查该目录是否有执行权限，目录下文件的删除或新建（写权限）需要执行权限的配合
2，设置的默认权限仅对在该目录下新建的文件才有效，对已存在的文件不生效。
3，改默认条目对目录本身不生效

### ACL的一些使用技巧

	[root@centos6 app]# getfacl test1 |setfacl --set-file=- test2   #复制test1 ACL给test2,test2之前的ACL被清空
ACL的备份与恢复

	[root@centos6 app]# getfacl -R dir   > acl.list  ##递归导出该目录下的所有ACL
	[root@centos6 app]# setfacl -R -b dir/    #清空该目录的ACL
	[root@centos6 app]# setfacl -R --set-file=acl.list dir   #恢复该目录下的ACL或者使用下面这条命令恢复

	[root@centos6 app]# setfacl --restore acl.list

- mask

在getfacl得到的条目中有一个是mask条目，所谓mask条目我理解为有效权限的意思，自定义的用户或者组的权限必须在mask的权限范围之内才会生效。

![](http://ot9scj6tc.bkt.clouddn.com/filemask.png)

当后来新增或更改的ACL条目的权限大于之前的mask范围时，mask会变大。

★★★ 当文件使用ACL功能后，ll列出的长选项中所在组的权限位不再表示该组的权限，而是表示该文件的mask值，使用chmod命令对组权限的更改也是作用于mask值上面。
		
		
	
