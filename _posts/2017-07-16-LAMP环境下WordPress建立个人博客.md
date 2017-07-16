每个IT工程师都期待拥有一个自己的博客站，本文讲述在CentOS 6系统LAMP环境下使用WordPress框架构建个人博客站的详细过程。


##----构建LAMP环境
###1，安装apache,mysql php
	yum install -y httpd mysql mysql-server php-mysql php php-devel php-gd php-xml

###2,设置服务自启动
	chkconfig --level 3  mysqld on
	chkconfig --level 3  httpd on
###3,启动服务
	/etc/init.d/httpd restart
	/etc/init.d/mysqld restart
###4，更改mysql数据库配置
	 #mysql_secure_installation
	Change the root password? [Y/n] y	#修改mysql管理员root密码
	New password:
	Re-enter new password:
	Password updated successfully!
	Remove anonymous users? [Y/n] y		#删除匿名用户
	Disallow root login remotely? [Y/n] n	#是否禁止root远程登录，选择否
	Remove test database and access to it? [Y/n] y		#移除test数据库
	Reload privilege tables now? [Y/n] y		#使当前更改立即生效
###5，新建wordpress数据库
	# mysql -uroot -p
	msql>create database wordpress;
	mysql> exit

##-------下载wordpress
[root@www tools]# wget https://cn.wordpress.org/wordpress-4.7.4-zh_CN.tar.gz
[root@www tools]# tar -zxvf wordpress-4.7.4-zh_CN.tar.gz
[root@www tools]# cp -r workspace/* /var/www/html/ 
[root@www tools]# cd /var/www/html/
[root@www html]# mv wp-config{-sample.php,.php}
[root@www html]# vim wp-config.php

##---安装完成。