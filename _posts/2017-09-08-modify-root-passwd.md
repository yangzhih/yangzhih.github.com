---
layout: post
title: "CentOs 修改root密码的正确姿势"
description: "CentOs 修改root密码的正确姿势"
category: articles
tags: passwd 
comments: false
---


## 场景

在生产或者日常使用中，偶尔会出现root密码忘记的情况。这时如何修改root密码成了一个工程师需要关注的小知识。下面详细说一下如何正确的修改root密码。

如果root密码丢失，远程操作使用正常办法是无法解决的。所以要修改root密码服务器必须在你身边。
修改root密码原理：在单用户模式下只加载bash而不进行身份验证。
CentOS 6版本和7版本进入单用户的方式略有不同，这里我把步骤分享给大家

进入单用户模式 --> 修改口令 --> 完成

## 实现步骤：

### Centos6版本

1 出现下面界面时按任意键打断默认启动
 
<img src="http://ot9scj6tc.bkt.clouddn.com/mrp1.png" width="600px"/>

2 选择对应内核，按e键进入编辑模式

<img src="http://ot9scj6tc.bkt.clouddn.com/mrp2.png" width="600px"/>
 
3 上下键选择kernel ，e键进入参数编辑模式
 
<img src="http://ot9scj6tc.bkt.clouddn.com/mrp3.png" width="600px"/>

4 在参数最后面添加一个 1 进入1启动级别 ，然后按esc返回上一级，b键启动

<img src="http://ot9scj6tc.bkt.clouddn.com/mrp4.png" width="600px"/>
 
5 进入单用户后更改密码
 
<img src="http://ot9scj6tc.bkt.clouddn.com/mrp5.png" width="600px"/>

### CentOS 7 版本

**方法1**（官方提供方法）

1 选择内核，e 键进入编辑模式

<img src="http://ot9scj6tc.bkt.clouddn.com/mrp6.png" width="600px"/>
 
2  在Linux16 对应行后加入`rd.break` 参数  Ctrl +x 启动
 
 <img src="http://ot9scj6tc.bkt.clouddn.com/mrp7.png" width="600px"/>

3 进入单用户后，使用mount选项，发现根分区以制度方式挂载

<img src="http://ot9scj6tc.bkt.clouddn.com/mrp8.png" width="600px"/>
 
4 重新挂载根分区，以rw方式。
执行` mount –o remount,rw /sysroot`

<img src="http://ot9scj6tc.bkt.clouddn.com/mrp9.png" width="600px"/>
 
5  切换根分区，修改root密码，创建打标签文件，并重启
 
<img src="http://ot9scj6tc.bkt.clouddn.com/mrp10.png" width="600px"/>

**方法2**：
在修改启动参数的时候指定初始化进程为bash,进入bash更改root密码， 同理，CentOS6也可以使用这种方式来修改密码(`rw  init=/bin/bash`)
步骤如下

1 进入编辑模式后，在Linux16 对应行添加 `rw init=/sysroot/bin/bash`，  Ctrl +x 启动
 
<img src="http://ot9scj6tc.bkt.clouddn.com/mrp11.png" width="600px"/>

2 因为是rw模式挂载，所以直接切根，修改密码，创建一个打标签文件，然后重启

 
<img src="http://ot9scj6tc.bkt.clouddn.com/mrp12.png" width="600px"/>


