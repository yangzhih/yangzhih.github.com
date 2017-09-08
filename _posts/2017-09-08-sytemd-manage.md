---
layout: post
title: "Sytemd管理"
description: "Sytemd管理"
category: articles
tags: [linux,systemd,init]
comments: false
---

## 写在前面


前文<a href="http://zhyang.top/articles/2017/09/01/Boot.html" >《开机流程分析》</a>中讲述在用户层阶段有对进程管理的工具有`systemd`和`init`两种。在`CentOS5,6`上面使用`init`作为进程管理的工具，在`CentOS7` 上面使用`sytemd`对进程进行管理，负责在系统启动或运行时，激活系统资源，服务器进程和其它进程。由于`init`对进程的管理方式以被人所熟知，所以本文主要讲述`sytemd`对进程的管理，以及二者的在使用上的异同。

**init局限性**：
+ 启动时间长，init进程是串行启动，只有前一个进程启动完毕，才会启动下一个进程。
+ 启动脚本复杂。init进程只是对执行进程启动脚本，脚本需要处理各种情况，进程如果有依赖性，脚本会变得非常复杂。

**Systemd的优缺点**：

systemd的设计目标是为系统的启动和管理提供一套完整的解决方案。systemd的功能非常强大，使用也相当方便。但是它体系过于庞大，结构也很复杂。在软件工程学中一直以高内聚，低耦合为目标，systemd的设计反而和操作系统的各个层面是一种强耦合的关系。也不太符合"keep simple, keep stupid"的Linux哲学。

**System的的新特性**：
+ 系统引导时实现服务并行启动
+ 按需启动守护进程
+ 自动化的服务依赖关系管理
+ 同时 采用socket 式与D-Bus 总线式激活 服务
+ 系统状态快照

>Systemd还有一个重要的特性是可以兼容init对进程的管理方式，降低了工程师的学习成本。

## 核心概念

unit 表示不同类型的`systemd` 对象，通过配置文件进行标识和配置；文件中主要包含了系统服务、监听socket 、保存的系统快照以及其它与init 相关的信息
>配置文件：
`/usr/lib/systemd/system`: 每个 服务最主要的启动脚本设置 ，类似于之前的`/etc/init.d/`
`/run/systemd/system`：系统执行过程中所产生的服务脚本，比上面目录优先运行
`/etc/systemd/system`：管理员建立的执行脚本，类似于`/etc/rc.d/rcN.d/Sxx`

**Unit分类**

	[root@centos7 ~]# systemctl -t help
	Available unit types:
	service
	socket
	busname
	target
	snapshot
	device
	mount
	automount
	swap
	timer
	path
	slice
	scope

## 启动相关

`sysVinit`中通常以`runlevel`的形式加载一组服务，在systemd启动过程中使用`target`取代`runlevel`，这也是启动过程中最大的变化。
`sysVinit`中定义了7种`runlevel`，0和6表示开机和重启，1级别只包含一些必须启动的服务，加载bash进程时不做身份校验，一般用于维护和紧急的救援工作。2345级别虽然有系统本身对应的级别说明，但是用户可以更改其相应的配置文件来做定制化的配置。在配置服务过程中，用户需要自己解决服务的依赖关系，比如如果你要使用web服务，需要网络服务的支持，所以web服务的启动需要在`network`服务之后。这种依赖关系的判断需要用户按照自己的经验来判断，如果判断失误非常容易造成无法启动的情况。
在`systemd`中默认使用了40多种`target`，当系统启动时，会根据配置文件来确定依赖关系并进行加载。下面是`runleve`l和`target`之间的对应表单：

	SysVinit runlevel              system target
	0                    runlevel0.target -> poweroff.target
	1                    runlevel1.target -> rescue.target
	2                    runlevel2.target -> multi-user.target
	3                    runlevel3.target -> multi-user.target
	4                    runlevel4.target -> multi-user.target
	5                    runlevel5.target -> graphical.target
	6                    runlevel6.target -> reboot.target

可以看出，在使用这5种`target`来对应`sysVinit`中的7种运行级别，从而实现向后的兼容性问题。
打开任意一个`target`配置文件，里面有一个非常重要的参数，关于依赖关系，即`Requires/Wants`关键字。`target`或者各个`unit`之间的依赖关系有该关键字来确认。其中`Requires`表示强制依赖关系，Wangts表示非强制依赖关系。除了`Wants`和`Requires`之外还可以有`WantedBy、RequiredBy、Conflicts、ConglictedBy、Before、After`等关键字来表示依赖与被依赖的关系。`systemd`实现自动化解决依赖关系也是靠这些关键字来实现。只有当依赖的[unit]全部激活后，当前`unit`才会被引导。每个关键字表示的含义如下：

	关键字                含义
	Requires           强制依赖关系
	Wants              非强制依赖关系，有依赖冲突时，暂时解除依赖关系
	Before             引导顺序，当前unit要在该值之前加载
	After              引导顺序，当前unit要在该值之后加载，与before相反
	Conflicts          冲突关系
	WantedBy           unit为service，弱依赖关系
	RequiredBy         unit为service，强依赖关系
	ConglictedBy      

上诉所说的依赖关系关键字都处于配置文件的`unit`小节中，`unit`小节中除了依赖关系之外还有`description`关键字表示描述信息。除了unit小节外还有一个`install`小节，表示当前unit是否开机需要加载，和`sysVinit`的`chkconfig  on` 设置开机自启动的功能是相同的。第一个被引导的target必须设置为`Alias=default.target`。

	Unit的类型为service时还有一个service小节，其中关键字也有很多，与服务启动相关。
	Type ：定义影响ExecStart 及相关参数的功能的unit 进程启动类型
	simple ：默认值，这个daemon 主要由ExecStart 接的指令串来启动后常驻于内存中
	forking ：由ExecStart 启动的程序透过spawns 延伸出其他子程序来作为此daemon 的主要服务。原生父程序在启动结束后就会终止
	oneshot ：与simple 类似，不过这个程序在工作完毕后就结束了，不会常驻在内存中
	dbus ：与simple 类似，但这个daemon 必须要在取得一个D-Bus的 的名称后，才会继续运作. 因此通常也要同时设定BusNname=才行
	notify ：在启动完成后会发送一个通知消息。还需要配合
	NotifyAccess 让 来让 Systemd  接收消息
	idle ：与simple 类似，要执行这个daemon 必须要所有的工作都顺利执行完毕后才会执行。这类的daemon 通常是开机到最后才执行即可的服务
	EnvironmentFile ：环境配置文件
	ExecStart ：指明启动unit 要运行命令或脚本的绝对路径
	ExecStartPre： ： ExecStart 前运行
	ExecStartPost： ： ExecStart 后运行
	ExecStop ：指明停止unit 要运行的命令或脚本
	Restart ：当设定Restart=1  时，则当次daemon 服务意外终止后，会再次自动启动此服务


明白上述配置文件所表示的含义，我想用户就可以自己动手来跟踪得出systemd的启动流程了。CentOS7用户层启动流程如下：

	内核初始化，执行initrd.target 所有单元，包括挂载/etc/fstab
	从initramfs 根文件系统切换到磁盘根目录
	systemd执行默认target 配置，配置文件/etc/systemd/system/default.target
	systemd 执行sysinit.target 初始化系统及basic.target 准备操作系统
	systemd 启动multi-user.target 下的本机与服务器服务
	systemd 执行multi-user.target 下的/etc/rc.d/rc.local
	Systemd 执行multi-user.target 下的getty.target 及登录服务
	systemd 执行graphical 需要的服务

**运行级别之间的切换**

	[root@centos7 ~]# init 3    兼容sysVinit命令 ,切换至对应的multi-user模式
	[root@centos7 ~]# systemctl isolate multi-user.target
	[root@centos7 ~]# systemctl get-default  获取当前默认启动的target
	[root@centos7 ~]# systemctl set-default multi-user.target  设置默认启动的target 或者使用如下命令
	[root@centos7 ~]# ln -sf /lib/systemd/system/multi-user.target  /etc/systemd/system/default.target


## Systemd常用命令实例

### 系统管理

Systemd把一些需要修改配置文件才能生效的功能也做成了新的命令，比如`hostnamectl,localectl,loginctl,timedatectl`等。

1，Systemctl命令，Systemd 的主命令，用于管理系统

	[root@centos7 system]# systemctl reboot   重启系统
	[root@centos7 system]# systemctl poweroff  关闭系统，切断电源
	[root@centos7 system]# systemctl halt  关机不断电源，相当于CPU不工作
	[root@centos7 system]# systemctl suspend  暂停系统
	[root@centos7 system]# systemctl rescue  进入救援模式（单用户）

Hostnamectl命令

	[root@centos7 system]# hostnamectl   显示当前主机信息
	[root@centos7 system]# hostnamectl set-hostname www.zhyang.top  设置主机名

Localectl命令

	[root@centos7 system]# localectl  查看系统当前字符集等信息
	[root@centos7 system]# localectl list-locales 查看系统支持的字符集信息
	[root@centos7 system]# localectl set-[keymap|locale|x11-keymap] … 设置系统使用的键盘布局，字符集等
	[root@centos7 system]# export LANG=zh_GN.gbk 使设置的字符集生效

Loginctl命令

	[root@centos7 ~]# loginctl list-sessions  列出当前会话
	[root@centos7 ~]# loginctl list-users  列出当前登录用户
	[root@centos7 ~]# loginctl show-user root 列出指定用户的登录信息

Timedatectl 命令

	[root@centos7 ~]# timedatectl [status]   列出当前时间详细信息
	[root@centos7 ~]# timedatectl list-timezones 列出所有可用时区
	[root@centos7 ~]# timedatectl set-timezones  Asia/Shanghai  设置当前时区
	[root@centos7 ~]# timedatectl set-time YYYY-MM-DD 设置时间
	[root@centos7 ~]# timedatectl set-time HH:MM:SS  
	[root@centos7 ~]# timedatectl set-ntp [yes|no]  开启或关闭ntp服务  

## UNIT管理

查看所有unit类型

	[root@centos7 system]# systemctl -t help


System 与sysVinit的服务对照表

	       Systemd                         sysVinit                    功能
	Systemctl start  [service]      service [service] start       立即运行某服务
	Systemctl stop  [service]       service [service] stop        立即停止某服务
	Systemctl enable  [service]     chkconfig [service] on        开机启动某服务
	Systemctl disable  [service]	chkconfig [service] on        开机不启动某服务


	[root@centos7 ~]# systemctl status  显示系统状态和每个unit状态（树形结构）
	[root@centos7 ~]# systemctl status  [unit_name] 指定unit的状态查看
	[root@centos7 ~]# systemctl -H 172.18.250.194 status httpd  远程查看某个主机的特定unit信息
	[root@centos7 ~]# systemctl list-units -t service --state active  查看当前已激活的所有服务
	[root@centos7 ~]# systemctl list-unit-files -t service --state [enabled]  查看系统开机启动的服务信息
	
	[root@centos7 ~]# systemctl kill [httpd.service] 杀死某个服务的所有子进程
	[root@centos7 ~]# systemctl daemon-reload 重载所有修改的配置文件
	[root@centos7 ~]# systemctl reload httpd.service 重载指定服务的配置文件
	[root@centos7 ~]# systemctl mask httpd.service 禁止手动自动启动某服务
	[root@centos7 ~]# systemctl unmask httpd.service 取消禁止



依赖关系

	[root@centos7 ~]# systemctl  show -p "Wants" multi-user.target   查看target非强制依赖的服务
	[root@centos7 ~]# systemctl  show -p "Requires" multi-user.target   查看target强制依赖的服务
	[root@centos7 ~]# systemctl list-dependencies httpd.service  列出一个unit的所有依赖
	[root@centos7 ~]# systemctl list-dependencies  --all  httpd.service  更详细列出所有依赖关系

## 实例：编译安装Nginx并编写systemd启动文本

	[root@centos7 ~]# wget http://nginx.org/download/nginx-1.9.4.tar.gz
	[root@centos7 nginx-1.9.4]# tar -xf nginx-1.9.4.tar.gz
	[root@centos7 nginx-1.9.4]# cd nginx-1.9.4/
	[root@centos7 nginx-1.9.4]# ./configure --prefix=/usr/local/nginx
	[root@centos7 nginx-1.9.4]# make
	[root@centos7 nginx-1.9.4]# make install
	[root@centos7 nginx-1.9.4]# cd /usr/lib/systemd/system
	[root@centos7 system]#cat >nginx.service<<EOF
	[Unit]
	Description=nginx - high performance web server
	Documentation=http://nginx.org/en/doc/
	#After 参数设置启动顺序
	After=network.target remote-fs.target nss-lookup.target
	
	[Service]
	Type=forking
	PIDFile=/usr/local/nginx/logs/nginx.pid
	ExecStartPre=/usr/local/nginx/sbin/nginx -t
	#ExecStartPre参数可确保ExecStart参数启动之前执行的命令，这里启动之前进行配置文件的正确性检测
	ExecStart=/usr/local/nginx/sbin/nginx	#启动Nginx服务
	ExecReload=/bin/kill -s HUP $MAINPID   #重载配置文件
	ExecStop=/bin/kill -s QUIT $MAINPID   #关闭Nginx服务
	PrivateTmp=true
	
	[Install]
	WantedBy=multi-user.target
	EOF
	[root@centos7 system]# systemctl daemon-reload 
	[root@centos7 system]# systemctl status nginx.service
	[root@centos7 system]# systemctl start nginx.service
	
