---
layout: post
title: "LAMP环境下WordPress建立个人博客"
description: "Guide to setup readonly mode for some users in django admin"
category: articles
tags: [python, django, django-admin]
comments: false
---



每个IT工程师都期待拥有一个自己的博客站，本文讲述在CentOS 6系统LAMP环境下使用WordPress框架构建个人博客站的详细过程。


## ----构建LAMP环境 ##


### 1，安装apache,mysql php ###

	yum install -y httpd mysql mysql-server php-mysql php php-devel php-gd php-xml

### 2，设置服务自启动

	chkconfig --level 3  mysqld on
	chkconfig --level 3  httpd on

### 3,启动服务

	/etc/init.d/httpd restart
	/etc/init.d/mysqld restart

### 4，更改mysql数据库配置

	#mysql_secure_installation
	Change the root password? [Y/n] y	#修改mysql管理员root密码
	New password:
	Re-enter new password:
	Password updated successfully!
	Remove anonymous users? [Y/n] y		#删除匿名用户
	Disallow root login remotely? [Y/n] n	#是否禁止root远程登录，选择否
	Remove test database and access to it? [Y/n] y		#移除test数据库
	Reload privilege tables now? [Y/n] y		#使当前更改立即生效

### 5，新建wordpress数据库

	# mysql -uroot -p
	msql>create database wordpress;
	mysql> exit

## -------下载wordpress
### 1，下载wordpress压缩包，并把压缩包拷贝至html目录
	[root@www tools]# wget https://cn.wordpress.org/wordpress-4.7.4-zh_CN.tar.gz
	[root@www tools]# tar -zxvf wordpress-4.7.4-zh_CN.tar.gz
	[root@www tools]# cp -r workspace/* /var/www/html/ 
	[root@www tools]# cd /var/www/html/
### 2，修改wordpress的配置文件中的数据库名称，以及用户密码，使之与之前的配置一致	
	[root@www html]# chmod -R 755 ./*     #修改wordpress权限为可读可执行
	[root@www html]# mv wp-config{-sample.php,.php}  #修改配置文件为指定wp-config.php
	[root@www html]# vim wp-config.php #修改配置文件的下列字段为上文新建数据库参数

	 /** WordPress数据库的名称 */
	 define('DB_NAME', 'database_name_here');
	 /** MySQL数据库用户名 */
	 define('DB_USER', 'username_here');
	 /** MySQL数据库密码 */
	 define('DB_PASSWORD', 'password_here');
	 /** MySQL主机 */
	 define('DB_HOST', 'localhost');
 

## -------安装完成。

到此为止博客个人博客站的创建已基本完成，打开web界面做简单的配置界面就可以开心的写自己的博客了。