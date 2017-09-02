---
layout: post
title: "boot分区和fstab文件被损坏后的解决方案"
description: "/boot 和/etc/fstab被损坏后的解决方案"
category: articles
tags: boot
comments: false
---

## 一 故障描述

	1  /boot分区被损坏
	2  /etc/fstab文件被误删除

由于/boot分区被损坏，所以导致grub无法引导操作系统。/etc/fstab文件被误删导致在进入Rescue模式原操作系统不会被自动挂载至/mnt/sysimg下。

## 二 解决思路

	1 首先恢复fstab文件，使救援模式能够识别原操作系统。
	2 恢复系统引导所需的grub文件
	3 恢复系统启动所需的内核文件和虚拟磁盘文件

## 三 故障判定

1 磁盘主引导记录被损坏
		现象：直接跳过磁盘启动，从其他启动项启动（光盘或者网络）
2 系统从磁盘启动，但停滞在如下阶段。
	Grub引导1_5阶段出错，导致无法进入grub的2阶段的启动菜单界面
	解决思路，修复grub

<img src="http://ot9scj6tc.bkt.clouddn.com/boot1.png" width="600px" />
 
## 四 故障解决流程

### 准备工作：
		操作系统版本相同的光盘1张，光驱1个
		
>解决流程

### 1 开机后选择从光驱引导
 <img src="http://ot9scj6tc.bkt.clouddn.com/boot2.png" width="600px" />
### 2 选择救援模式
 
<img src="http://ot9scj6tc.bkt.clouddn.com/boot3.png" width="600px" />
### 3 选择语言

默认选择English

 <img src="http://ot9scj6tc.bkt.clouddn.com/boot4.png" width="600px" />
### 4  选择键盘类型
一般默认选择，特殊键盘选择指定类型即可

 <img src="http://ot9scj6tc.bkt.clouddn.com/boot5.png" width="600px" />


### 5 配置网络
此故障无需配置网络，选择NO

 <img src="http://ot9scj6tc.bkt.clouddn.com/boot6.png" width="600px" />

### 6 选择开启救援模式
开启救援后，会试图把原来的Linux系统挂载在/mnt/sysimage目录下
 <img src="http://ot9scj6tc.bkt.clouddn.com/boot7.png" width="600px" />
 <img src="http://ot9scj6tc.bkt.clouddn.com/boot8.png" width="600px" />

由于原来系统下没有/etc/fstab文件，所以救援模式无法挂载分区至/mnt/sysimg下
回车，开启一个shell命令行
 <img src="http://ot9scj6tc.bkt.clouddn.com/boot9.png" width="600px" />
 
### 7 查看原来系统所在磁盘分区
使用fdisk 命令查看原来系统所在磁盘的分区详细信息。

  <img src="http://ot9scj6tc.bkt.clouddn.com/boot10.png" width="600px" />

从结果可以看出，/dev/sda1的boot字段有*标识，表示该分区为boot分区；
/dev/sda5的分区类型为swap，表示该分区为swap分区；
其他分区从fdisk的返回结果暂时无法确定其挂载点是什么。
创建一个临时目录，挂载无法判断的分区，通过内容判断其挂载点

  <img src="http://ot9scj6tc.bkt.clouddn.com/boot11.png" width="600px" />

挂载/dev/sda2，查看其目录结构符合根分区的目录结构，所以判断该分区为根分区

  <img src="http://ot9scj6tc.bkt.clouddn.com/boot12.png" width="600px" />

### 8 编辑一个fstab文件
在原系统的根分区下编辑一个fstab文件，路径为/mnt/tmp/etc/fstab

**注意  救援模式不支持使用vim,请使用vi编辑器添加下图内容**

  <img src="http://ot9scj6tc.bkt.clouddn.com/boot13.png" width="600px" />

### 9  重启系统，重新进入救援模式
  <img src="http://ot9scj6tc.bkt.clouddn.com/boot14.png" width="600px" />

选择语言，键盘类型，配置网络
在Rescue界面选择Continue后，显示原来系统已挂载至/mnt/sysimg下
 
 <img src="http://ot9scj6tc.bkt.clouddn.com/boot15.png" width="600px" />

### 10 恢复boot目录内容

	
	sh-4.1# chroot /mnt/sysimage     #切换根分区
	sh-4.1# mount /dev/cdrom /mnt   	#挂载光盘
	sh-4.1# rpm –ivh /mnt/Packages/kernel-2.6.32-696.el6.x86_64.rpm --force 	#重新安装内核	
	sh-4.1# grub-install /dev/sda   #修复grub

具体过程可见下图

  <img src="http://ot9scj6tc.bkt.clouddn.com/boot16.png" width="600px" />

### 11 创建grub.conf文件

在/boot/grub下创建一个grub.conf文件，内容至少包括下图几项
	default ：若启动菜单为多个，默认从哪个启动
	timeout：在达到时间无指令打断后默认启动
	title:菜单内容，用户可自定义
	kernel: 内核文件路径
	initrd: 虚拟文件系统路径

>注意 kernel 和initrd的路径都是以/boot为根
>即：/boot=/

	eq: /boot/vmlinuz-2.6.32-696.el6.x86_64  =  /vmlinuz-2.6.32-696.el6.x86_64

  <img src="http://ot9scj6tc.bkt.clouddn.com/boot17.png" width="600px" />

### 12 退出当前根环境，重启系统
 
 <img src="http://ot9scj6tc.bkt.clouddn.com/boot18.png" width="600px" />

系统引导之后会出现Selinux的检查阶段，耗时会很久，请耐心等待
 
  <img src="http://ot9scj6tc.bkt.clouddn.com/boot19.png" width="600px" />

 <img src="http://ot9scj6tc.bkt.clouddn.com/boot20.png" width="600px" />