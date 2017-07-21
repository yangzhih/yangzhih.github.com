---
layout: post
title: "Linux系统的账号管理"
description: "Linux系统的账号管理"
category: articles
tags: [linux, 账号管理]
comments: false
---

## 写在前面
&nbsp;&nbsp;&nbsp;&nbsp;作为一个运维工程师来讲，系统下的账号管理是工作中很重要的一个环节，所以了解系统的账号管理还是非常有必要的。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;首先我们要知道对于Linux系统而言，它并不知道你是谁，它仅识别用户的ID号，而用户想要登录Linux系统，必须取得两个ID号(uid，gid),用户的名称与uid的映射关系就存在/etc/passwd。用户想要登录Linux系统，需要两种id，UID和GID（组id）,在登录界面，用户输入登录名后一系统首先会判断passwd文件中是否有该用户，没有就跳出，如果有的话就会取出用户的uid，gid等信息，其中最重要的取出用户的shell，判断该shell是否要登录，若可登录再去匹配用户密码，相匹配成功才会登录成功。一个简单登录操作背后有着很繁杂的过程。在用户管理主要有四个配置文件/etc/passwd,/etc/shadow,/etc/group,/etc/gshadow。详细了解这四个文件的配置以及各字段的含义是深入了解Linux账号管理的前提条件。下面笔者写下自己的了解，也许笔者了解的并不完善，可使用 man 5 [config_file]命令查看文档。

### /etc/passwd 文件详解

![](http://ot9scj6tc.bkt.clouddn.com/passwd.png)

passwd配置文件每行为一个账号记录使用“:”作为分隔符分为七个字段，每个一字段含义分别为【account:password:UID:GID:GECOS:directory:shell】<br/>

1，account 账户名称。<br/>
2，密码，在Linux系统发展初期，密码放在该字段，由于该文件每个用户都可读，出于安全方面的考虑，把加密后的密码存在shadow中。在该使用X作为密码占位符。（可以使用pwunconv，pwconv两个命令在两种密码存储方式之间进行切换，做完实验后恢复到当前状态）。<br/>
3，用户在系统下的标识符，在Linux系统下用户可分为三种：管理员，系统用户，一般用户。<br/>
   &nbsp;&nbsp;&nbsp;通过uid判断，uid=0时为管理员；<br/>
   &nbsp;&nbsp;&nbsp;CentOS6和以前版本，uid在1-500之间，为系统用户；CentOS7版本中，uid在1-1000之间，为系统用户。<br/>
   &nbsp;&nbsp;&nbsp;CentOS6和以前版本，uid大于等于500，为一般用户；CentOS7版本中，uid大于等于1000，为一般用户。<br/>
   &nbsp;&nbsp;&nbsp;由于系统根据uid识别用户类型，所以不要轻易改变passwd文件，尤其是管理员用户。<br/>
4，用户组id,该ID号用户和/etc/group文件相关，对应于组名和gid。<br/>
5，用户的描述信息。<br/>
6，用户家目录。<br/>
7，用户使用的shell类型，当用户shell为/sbin/nologin时，用户不能登录。<br/>

### /etc/shadow 文件详解


&nbsp;&nbsp;&nbsp;&nbsp;在Linux系统早期设计中并没有shadow文件，账号密码存于passwd的第二个字段，因为很多程序运行需要读取passwd文件，每个用户都必须对该文件用有读权限，虽然密码已经经过加密，但仍然被暴力破解。所以把密码通过加密后存于shadow文件中，这样不仅安全，也方便了用户口令的管理。下面看看shadow文件的文件结构。

![](http://ot9scj6tc.bkt.clouddn.com/shadow.png)

&nbsp;&nbsp;&nbsp;&nbsp;shadow文件与passwd相同，使用“:”进行分割各个字段，shadow文件有9个字段，各字段分别代表【账号名称：口令：最近更改口令的日期：口令不可变更的时间：口令需要重新更改的时间：警告更改口令的时间：口令过期后的宽限时间：账号失效时间：无意义(保留)】<br/>
1，账号名称。<br/>
2，用户的加密口令，用户可以使用authconfig --passalgo=[sha256] -- update命令更改口令加密方式。<br/>
3，用户口令最后更改的时间。本时间是从1970年1月1日到更改密码的天数，1970年1月1日是Linux元年(Linux历史小知识)。<br/>
4，口令不可更改的时间（以第三字段为基准），如果是0表示口令随时可被更改，若是n，表示在更改口令后n天内不能更改口令。<br/>
5，口令需要更改的时间(以第三字段为基准)，该字段表示在用户更改口令后n天后需要重新更改口令，否则口令将失效。<br/>
6，警告更改口令的时间（以第5字段为基准），这个字段表示在用户需要更改口令的时间之前的n天开始提示警告用户更改口令。<br/>
7，口令过期后的宽限时间（以第5字段为基准），在口令过期后，用户在这个时间内可以登录系统，但是必须更改口令才能登录成功。<br/>
8，账号失效日期，与第三字段相同，从1970开始计算天数，到期后，账户将失效，该字段优先级大于口令过期日期，到期后，账户不可使用。<br/>
9，保留。<br/>

### /etc/group  /etc/gshadow 文件简述
在了解完passwd和shadow配置文件之后，我想对于理解group和gshadow文件已经是什么难事。这里就简述一下两个文件的文件结构含义。<br/>
group配置文件也是每行表示一条记录，以“:”作为分隔符，各个字段的含义为：【group_name:passwd:GID:user_list】<br/>
gshadow文件各字段含义如下【group_name:passwd:group_adminster:members】<br/>
这里有两个值得注意的地方：1，group的用户列表中默认只保存以该组为附加组的成员；2，gshadow文件的members字段必须与group的user_list一致。<br/>


只有简单的了解了上面的四个配置文件，才能更好的了解Linux系统的账号管理的功能，下面就来讲讲Linux系统的账号管理，我把该功能分为三个部分来讲，分别为用户管理，口令管理和用户组管理。
## 用户管理 