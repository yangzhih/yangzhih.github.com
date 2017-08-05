---
layout: post
title: "Linux软件包管理"
description: "Linux软件包管理"
category: articles
tags: [linux, rpm,yum]
comments: false
---

### 写在前面

在Linux的学习过程中，编译安装的软件管理方式使得很多新手对Linux望而却步。毕竟不是每个人都会进行源码编译的。当然这个问题对于Linux发行厂商而言也是阻止Linux推广的障碍。如果有一种机制能够在具有相同硬件环境和系统环境上面把软件事先编译好，然后加上与Windows类似的软件管理机制的话，Linux的软件管理就会简单很多，RPM和YUM（笔者是的环境是CentOS，其他发行版的软件管理机制就不多做介绍了）也就应运而生了。

## Linux软件包管理——RPM

### RPM是什么？
RPM(RedHat Packets Manager)，顾名思义，该软件管理机制是Redhat公司开发的。其最大的特点就是事先将你要安装的软件编译过，并且打包成为RPM机制的安装包，在安装过程中，会根据自己数据库里面的记录来判断安装的软件是否满足依赖性的关系，若满足就予以安装，不满足的话，用户要手动解决依赖关系后在进行安装。因为RPM有自己的数据库记录，所以软件的升级，查询，安装以及卸载都很方便。

### RPM使用方法
RPM的使用就是使用rpm命令对软件进行管理，包括对软件的安装，卸载，升级和查询。

- **软件安装**

在对软件安装之前要获取rpm格式的软件包，并和自己的硬件平台相匹配。下面是对软件包的命名详解图。

![](http://ot9scj6tc.bkt.clouddn.com/rpm.png)

从系统安全方面考虑，安装软件是一件风险非常大的事情，所以需要root用户的权限才可以。使用rpm -i命令进行安装软件。以安装vsftpd相关的软件为例。

	[root@centos7 Packages]# rpm -i vsftpd-3.0.2-21.el7.x86_64.rpm    
	[root@centos7 Packages]# rpm -ivh vsftpd-sysvinit-3.0.2-21.el7.x86_64.rpm  
	Preparing...                          ################################# [100%]
	Updating / installing...
	   1:vsftpd-sysvinit-3.0.2-21.el7     ################################# [100%]
	#  i选项，i是install的缩写，i表示安装，不讲进度参数的话，默认安装不打印信息
	#  v查看详细的安装进度，可以使用vv选项更加详细，h选项是以信息栏的方式显示安装进度
	#  rpm 安装软件时也可同时安装多个软件，或者进行网络安装，只要提供具体的rpm软件包路径

 RPM虽然在很大程度上解决了软件管理的难度，但是也有不可忽视的缺点，其对依赖关系的处理上就不是特别擅长，需要管理员自己去解决依赖关系。所以在安装软件的时候或者在安装之前管理员就发现了问题，这个时候下面参数就可以解决相关问题。

	--nodeps  忽略依赖关系；使用该参数会有危险性，软件的依赖性是软件自身的运行机制或者提供的功能导致的，如果忽略，会发生未知错误
	--replacefiles 覆盖，在安装时有时会抛出某个文件已安装或者版本不合的错误，可以使用该参数覆盖文件。覆盖的操作是无法回退的，读者慎用
	--replacepkgs   重现安装某软件
	--force   强制安装，就是上面两个参数的综合体
	--test   测试安装，用于测试该软件是否可以安装在使用的Linux环境，也可以检查依赖关系
	--noscripts 如果读者在安装某个软件时，不想其在安装过程中执行自带的一些脚本可以使用该参数，但不建议，有时会安装失败或功能无法使用的情况。

在安装rpm软件包时的参数还有很多，笔者不能详录，也不常用。笔者建议使用ivh安装就可以了。

- **软件升级**

RPM包的升级方式尤其简单，读者如果要升级某个软件包，可以到各大站点下载版本高的软件包，使用-Uvh或者-Fvh升级即可。

	-Uvh   如果后面跟的软件包未安装过，自动安装；如果本机的软件是旧版就更新。
	-Fvh  该参数只会更新在系统上面已安装的软件包，若为安装，不会升级。

升级的参数就是以上两个，读者可以根据自己所需，选择合适的参数，在做大批量的软件升级时，笔者建议使用-Fvh参数。

- **软件查询**

在对软件管理过程中，管理通常会想查询某个软件包是否已经安装在系统中，或者查询该软件的详细的信息等，这些问题RPM都可以解决。下面做一下简单的介绍

	rpm -q[a][p][licdRf] packetname or packetpath

	rpm -q 后面可以跟软件包的名称，查询某个软件是否已安装
		-qa  查询系统中已经安装的所有软件
		-ql  接软件名，查询该软件的所有文件列表
		-qi  接软件名，查询该软件的详细信息
		-qc  接软件名，查询该软件的配置文件(config)
		-qd  接软件名，查询该软件的文档所在的路径
		-qR  接软件名，查询该有关的依赖软件的相关文件信息
		
		-qf  接(配置)文件名，查询该文件属于某个软件包

		-qp[licdR]  接软件包所在路径，加后面参数查询相关信息，参数和上面功能一致

>实例

![](http://ot9scj6tc.bkt.clouddn.com/chaxun.png)


- **软件卸载**

软件卸载也尤其简单就一个e选项，但是在软件卸载过程中也会遇到依赖关系的问题，所以在进行软件卸载时一定要从最上层向底层卸载才不容易发生问题，当然读者可以使用--nodeps选项来忽略依赖关系，但这种操作可能会带来未知错误，所以要慎用。

	[root@centos7 Packages]# rpm -e vsftpd
	[root@centos7 Packages]# rpm -e --nodeps  packetname

- **杂项**

>重建数据库

有时候在误操作的的时候会把rpm的数据库文件破坏掉，这时我们可以重建，使用rpm --rebuilddb,但是重建的数据库是不包含之前的软件信息的，所以有和没有没什么区别。

>验证

当系统发生异常时有时会做各种检测，这时或许要考虑软件是否被篡改，或者被黑客替换某些文件，可以使用rpm -V 接软名的方式查看是否有文件被篡改，看这些改动是否是正常的改动或者异常的，一般被正常改动的多为配置文件，若是二进制文件被改动，就要特别关注了。

>公钥文件

验证可以发现程序是否异常，但是如果安装的软件本身就被篡改过那岂不是风险更大？为了解决这些问题，Linux发行的镜像中都存在一个公钥文件（RPM-GPG-KEY-CentOS-6），也称为数字证书，读者在安装软件时也可以安装这个数字证书，使用rpm --import  RPM-GPG-KEY-CentOS-6命令安装证书，卸载使用-e,查看证书信息使用rpm -qi `rpm -qa|grep pubkey`的方式。    

## Linux软件包管理——YUM

我想每一个熟悉RPM包管理的读者在使用RPM过程中肯定遇到过遇到依赖死锁束手无策的情况，有时候想安装一个软件A，但是安装A就需要软件B的支持，安装软件B有需要软件C的支持，当安装软件C时，系统又告诉你需要软件A的支持，当依赖关系出现了环状关系，只能使用rpm的nodeps选项或者强制安装。但是这样的方式总会有一些莫名奇妙的小问题。这个时候就体现到YUM包管理器的重要性了。
笔者想通过三个小部分来讲述YUM软件包管理器的功能与配置的问题，分别为YUM的定义，YUM的功能，YUM客户端的配置。希望读者能从此文有丁点收获。

###YUM是什么？
YUM（Yellow dog Updater,Modified）是一个在红帽系列Linux发行版的一个软件包管理器。YUM包管理器是基于rpm的包管理器，yum通过分析rpm包的标题数据信息，根据其软件的相关性来判断出依赖关系的解决方案，然后自动处理软件的依赖属性问题。可以高效的解决软件管理方面的疑难问题。

###YUM的功能
YUM的功能与通用的包管理器大致相同，软件的查询，安装，升级，卸载。除了这些之外还有软件包组管理的的功能。

- **软件安装/升级**

使用yum安装或者升级软件非常简单， 使用yum install/reinstall/update packetname 命令，以安装lftp软件为例

		[root@centos6 workspace]#yum -y install lftp     #使用YUM安装ftp客户端软件  reinstall参数为重新安装软件
		Loaded plugins: fastestmirror, refresh-packagekit, security
		...
		lftp-4.0.9-14.el6.x86_64.rpm                                    | 755 kB     00:00     
		...
		Installed:
		  lftp.x86_64 0:4.0.9-14.el6                                                           
		Complete!


从上面显示可以看到使用YUM对软件的安装是非常快速的，读者无需考虑软件平台是否支持该包，更无需解决复杂的依赖关系。从这里就能出使用YUM安装软件的便利。
		
- **查询功能**

yum的查询功能很丰富，使用方法是 yum [list|info|search|provides|whatprovides|repolist]，下面笔者为大家讲述一下他们的具体含义与简单用法，由于显示很多信息，笔者会把非关键信息以省略的方式展示。

	[root@centos6 workspace]#yum list    #列出服务器上面提供的所有软件包，包括软件包，版本，安装方式（或者仓库名）
	[root@centos6 workspace]#yum list updates  #查看系统内那些包可以升级  
	[root@centos6 workspace]#yum list installed  #查看系统内那些包以安装
	[root@centos6 workspace]#yum list available  #查看系统内有哪些包可以在这个yum仓库安装
	[root@centos6 workspace]#yum info lftp   #查看yum仓库内某个软件的详细信息(安装状态，大小以及一些协议信息)
	[root@centos6 ~]#yum search lftp	#搜索lftp（可以是关键字）相关的软件有哪些，以冒号分隔软件名和软件描述信息
	[root@centos6 ~]#yum whatprovides /etc/ssh/sshd_config  #查看指定的特性（配置文件）属于哪个软件
	[root@centos6 ~]#yum provides passwd


- **软件卸载**

YUM的软件卸载很简单，只有一个remove操作，后面可以跟多个软件包或者通配符，无需考虑依赖关系。
	
	[root@centos6 ~]#yum remove  lftp   #卸载某个软件包

- **组和仓库相关操作**

与其他包管理器不同的就是yum有对包组管理的的功能。

	[root@centos6 ~]#yum repolist   # 查看本机有哪些yum仓库
	[root@centos6 ~]#yum clean all   #清空yum缓存
	[root@centos6 ~]#yum makecahe	#构建yum缓存
	[root@centos6 ~]#yum history  #yum的事物历史
	[root@centos6 ~]#yum history [info|list|packages-list|packages-info|rollback|new|sync|status|redo|undo]
	#这个为yum的一些高级功能，也不常用，读者可以使用man帮助来学习。
	[root@centos6 ~]#yum grouplist  # 查看系统拥有哪些软件包组
	[root@centos6 ~]#yum groupinstall  "development tools"  #安装某个软件包组
	[root@centos6 ~]#yum groupupdate  "development tools"   #更新某个软件包组
	[root@centos6 ~]#yum groupremove  "development tools"   #卸载某个软件包组
	[root@centos6 ~]#yum groupinfo  "development tools"     #查看某个软件包组的信息
	

>***yum 的命令行选项***

	yum [options] [command] [package ...]
		--nogpgcheck ：禁止进行gpg check
		-y:  自动回答为“yes”
		-q ：静默模式
		--disablerepo=repoidglob ：临时禁用此处指定的repo
		--enablerepo=repoidglob ：临时启用此处指定的repo
		--noplugins ：禁用所有插件

## 配置yum仓库

- **配置yum客户端**



在对yum的客户端进行配置之前，首先要了解一下它都有哪些配置。yum的配置文件有多个，本文只江苏其中重要的两个/etc/yum.conf ，/etc/yum.repos.d/目录。打开yum.conf文件之后可以看到一下几个参数

	cachedir  #缓存路径
	keepcache  #yum安装的时下载的软件包是否删除
	logfile  #yum日志路径
	gpgcheck=1  #检查秘钥
	installonly limit  # 最多同时安装几个包
	...

yum仓库的客户端的配置文件在/etc/yum.repos.d/目录下，必须以.repo为后缀来命名。读者可以把系统默认的删除或者备份在其他路径之下。通常在生产环境中需要在本地局域网搭建yum仓库所以学会如何配置yum源是一个很重要的技能

	[cdrom]                                                                            
	name=cdrom
	baseurl=http://192.168.0.1/centos/$releasever/
	gpgcheck=1
	gpgkey=http://192.168.0.1/centos/$releasever/RPM-GPG-KEY-CentOS-$releasever

其中name为yum仓库的名称（自定义），baseurl为yum仓库的路径，这个路径很重要，该路径为repodata目录所在的路径。yum能够完美的解决依赖关系就是靠repo这个目录下的文件，该目录下中的数据是yum在分析RPM包的标题数据后所产生的数据。因此，baseurl的路径下一定有一个repo的目录存在。gpgcheck和yum.conf中含义相同，该参数最后生效，如果该参数为0表示在使用该yum仓库的软件时不检查数字证书。gpgkey为数字证书的存在路径。


- **配置yum客户端**

服务器端的配置也很简单，读者需要把安装光盘的文件或者第三方的文件中的软件包拷出来，放在同一目录下，执行createrepo packages（包所在目录）命令来生成repodata目录然后把数字证书放入该目录。最简单有效的方法是把光盘中的文件全部拷出来。
当然要配置yum服务端需要搭建一个简单的服务器，可以使用web服务器或者ftp服务器然后把配置好的yum仓库置于该种服务器软件所对应的目录下。如果是ftp服务器搭建yum仓库，客户端的配置baseurl应该是使用ftp协议，如果是在本机配置使用file://后跟绝对路径。