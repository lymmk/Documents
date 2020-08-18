# 更换 yum 源 #
1. 备份原数据
	> 	# cd /etc/yum.repos.d/
	> 	# mv CentOS-Base.repo CentOS-Base.repo.bak

2. 下载阿里云源
	> 	# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

3. 更新生效
	>  	# yum clean all
	>  	# yum makecache
	>     # yum -y update

# 配置 FTP #
1. 安装 vsftpd
	> 	 # 安装 yum install -y vsftpd
	> 	 # 设置开机启动 systemctl enable vsftpd.service
	> 	 # 启动 systemctl start vsftpd.service
	> 	 # 停止 systemctl stop vsftpd.service
	> 	 # 查看状态 systemctl status vsftpd.service

2. 配置 FTP
	  	
	> 	# 修改配置文件
	> 	
	> 	vim /etc/vsftpd/vsftpd.conf
	> 	
	> 	#显示行号
	> 	:set number
	> 	
	> 	#修改配置 12 行
	> 	anonymous_enable=NO
	> 	
	> 	#修改配置 33 行
	> 	anon_mkdir_write_enable=YES
	> 	
	> 	#修改配置48行
	> 	chown_uploads=YES
	> 	
	> 	#修改配置72行
	> 	async_abor_enable=YES
	> 	
	> 	#修改配置83行
	> 	ascii_upload_enable=YES
	> 	
	> 	#修改配置84行
	> 	ascii_download_enable=YES
	> 	
	> 	#修改配置87行
	> 	ftpd_banner=Welcome to blah FTP service.
	> 	
	> 	#添加下列内容到vsftpd.conf末尾
	> 	use_localtime=YES
	> 	listen_port=21
	> 	idle_session_timeout=300
	> 	guest_enable=YES
	> 	data_connection_timeout=1
	> 	virtual_use_local_privs=YES
	> 	pasv_min_port=40000
	> 	pasv_max_port=40010
	> 	accept_timeout=5
	> 	connect_timeout=1allow_writeable_chroot=YES

1. 建立用户文件
	> 	# 创建编辑用户文件
	> 	vim /etc/vsftpd/virtusers
	> 	# 第一行为用户名，第二行为密码。不能使用root作为用户名 
	> 	mkftp
	> 	liyan8023
1. 生成用户数据文件
	> 	db_load -T -t hash -f /etc/vsftpd/virtusers /etc/vsftpd/virtusers.db
	> 
	> 	#设定PAM验证文件，并指定对虚拟用户数据库文件进行读取
	> 
	> 	chmod 600 /etc/vsftpd/virtusers.db 
1. 修改 /etc/pam.d/vsftpd 文件
	> 
	> 	# 修改前先备份 
	> 	cp /etc/pam.d/vsftpd /etc/pam.d/vsftpd.bak
	> 	vi /etc/pam.d/vsftpd
	> 	#先将配置文件中原有的 auth 及 account 的所有配置行均注释掉
	> 	auth sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/virtusers 
	> 	account sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/virtusers 
	> 	# 如果系统为32位，上面改为lib

1. 新建用户，并指定目录

	> 	#用户登录终端设为/bin/false(即：使之不能登录系统)
	> 	useradd mkftp -d /home/mkftp -s /bin/false
	> 	chown -R mkftp:mkftp /home/mkftp	

1. 建立虚拟用户个人配置文件

	> 	mkdir /etc/vsftpd/vconf
	> 	cd /etc/vsftpd/vconf
	> 	
	> 	#这里建立虚拟用户mkftp配置文件
	> 	touch mkftp
	> 	#编辑leo用户配置文件，内容如下，其他用户类似
	> 	vi mkftp
	> 	
	> 	local_root=/home/mkftp
	> 	write_enable=YES
	> 	anon_world_readable_only=NO
	> 	anon_upload_enable=YES
	> 	anon_mkdir_write_enable=YES
	> 	anon_other_write_enable=YES
	> 	#建立mkftp用户根目录
	> 	mkdir -p /home/mkftp

1. 防火墙设置

	> 	IPtables 的设置方式：
	> 	
	> 	vi /etc/sysconfig/iptables
	> 	# 编辑iptables文件，添加如下内容，开启21端口
	> 	-A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT
	> 	-A INPUT -m state --state NEW -m tcp -p tcp --dport 40000:40010 -j ACCEPT
	> 	
	> 	firewall 的设置方式：
	> 	
	> 	firewall-cmd --zone=public --add-service=ftp --permanent
	> 	firewall-cmd --zone=public --add-port=21/tcp --permanent
	> 	firewall-cmd --zone=public --add-port=40000-40010/tcp --permanent

1. 重启 vsftpd 服务

	> 	systemctl restart vsftpd.service

1. 使用ftp工具连接测试

	> 	# yum install ftp 
	> 	# ftp 192.168.31.121

	出现 500、503 、200 等问题时，关闭SELINUX试试。
	
	> 	#打开SELINUX配置文件
	> 	vim /etc/selinux/config
	> 	
	> 	#修改配置参数
	> 	#注释  
	> 	SELINUX=enforcing
	> 	
	> 	#增加  
	> 	SELINUX=disabled	
	> 	
	> 	#修改完成后，需要重启！

	如果还不行，手动关闭防火墙。
	
	> 	systemctl stop firewalld.service

# 升级 Python #

1. 安装相应的编译工具
	> yum -y groupinstall "Development tools"
	> 
	> yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
	> 
	> yum install -y libffi-devel zlib1g-dev
	> 
	> yum install zlib* -y
	
1. 处理安装包

	> 	# wget wget https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tar.xz
	> 	# tar -xvJf  Python-3.7.2.tar.xz
	> 	# mkdir /usr/local/python3
	> 	# cd Python-3.7.2

1. 配置编译
	> 
	> 	# ./configure --prefix=/usr/local/python3 --enable-optimizations --with-ssl 
	> 	# make && make install

1. 创建软链接，替换Python2.7

	> 	# rm -rf /usr/local/bin/python
	> 	# ln -s /usr/local/python3/bin/python3 /usr/local/bin/python
	> 	# ln -s /usr/local/python3/bin/pip3 /usr/local/bin/pip3
1. 解决 **yum** 无法使用问题

	> 
	> 	# vi /usr/bin/yum
	> 	# #!/usr/bin/python --#!/usr/bin/python2
	> 
	> 	# vi /usr/libexec/urlgrabber-ext-down 
	> 	# #!/usr/bin/python --#!/usr/bin/python2

# 安装 Mysql5.7 #
1. 卸载默认数据库

	> 	# rpm -qa | grep mariadb
	> 	# yum -y remove mari*
	> 	# rm -rf /var/lib/mysql/*
	> 	# rpm -qa | grep mariadb
	> 	# find / -name mysql 查看残余目录和文件并删除
1. 安装YUM Repo

	> 	# wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
	> 	# rpm -ivh mysql57-community-release-el7-9.noarch.rpm

1. 安装
	> 	# cd /etc/yum.repos.d/
	> 	# yum install mysql-server
	> 	
	> 	# 安装 pip install mysqlclient 需要
	> 	# yum install mysql-devel
	> 	
	> 	# 启动服务
	> 	# systemctl start mysqld
	> 	# 查看运行状态
	> 	# systemctl status mysqld 

1. 登陆

	> 	# grep 'temporary password' /var/log/mysqld.log 
	> 	# 2020-07-27T03:12:21.731840Z 1 [Note] A temporary password is generated for root@localhost: S<Sh3g!1(1NA
	> 	# S<Sh3g!1(1NA  即为 root 用户默认登录密码

	> 	# mysql -uroot -p
1. 修改密码

	> 	# 登录之后需要先修改密码，否则基本不能进行任何操作
	> 	# ALTER USER 'root'@'localhost' IDENTIFIED BY 'xxxx'; 

	> 	# 修改密码策略
	> 	# SHOW VARIABLES LIKE 'validate_password%';
	> 	# 改为最低级别密码策略
	> 	# set global validate_password_policy=LOW;

1. 开启远程登录

	> 	# use mysql
	> 	# update user set Host='%' where User='root'; 
	> 	# 或者
	> 	# grant all privileges on (database)*.(table)* to (user)root@(host)"%" identified by (passwod)"xxxx";
	> 	# flush privileges;

1. 关于 MySql 密码策略

	> 	# validate_password_length  固定密码的总长度；
	> 	
	> 	# validate_password_dictionary_file 指定密码验证的文件路径；
	> 	
	> 	# validate_password_mixed_case_count  整个密码中至少要包含大/小写字母的总个数；
	> 	
	> 	# validate_password_number_count  整个密码中至少要包含阿拉伯数字的个数；
	> 	
	> 	# validate_password_policy 指定密码的强度验证等级，默认为 MEDIUM；
	> 		LOW：只验证长度；
	> 		MEDIUM：验证长度、数字、大小写、特殊字符；
	> 		STRONG：验证长度、数字、大小写、特殊字符、字典文件；
	> 	
	> 	# validate_password_special_char_count 整个密码中至少要包含特殊字符的个数；
1. 其他配置

	> 	# 设置安全选项：
	> 	mysql_secure_installation
	> 	
	> 	# 关闭MySQL
	> 	systemctl stop mysqld 
	> 	
	> 	# 重启MySQL
	> 	systemctl restart mysqld 
	> 	
	> 	# 查看MySQL运行状态
	> 	systemctl status mysqld 
	> 	
	> 	# 设置开机启动
	> 	systemctl enable mysqld 
	> 	
	> 	# 关闭开机启动
	> 	systemctl disable mysqld 
	> 	
	> 	# 配置默认编码为utf8
	> 	vi /etc/my.cnf
	> 	[mysqld]
	> 	character_set_server=utf8
	> 	
	> 	# 修改默认数据存储位置
	> 	systemctl stop mysqld
	> 	mv /var/lib/mysql /usr/
	> 	vi /etc/my.cnf
	> 	[mysqld]
	> 	datadir=/usr/mysql 
	> 	socket=/usr/mysql/mysql.sock
	> 	
	> 	[client]
	> 	socket=/usr/mysql/mysql.sock
	> 	
	> 	[mysql]
	> 	socket=/usr/mysql/mysql.sock
	> 	
	> 	systemctl start mysqld

# 配置 Apache #
1. 安装 httpd 服务
	> 	# yum install httpd
	> 	
	> 	# 开机启动
	> 	# /sbin/chkconfig httpd on
	> 	# 启动服务
	> 	# /sbin/service httpd start
	> 	# 查看运行状态
	> 	# systemctl status httpd

1. 配置防火墙

	> 	# 开启 80 端口
	> 	# firewall-cmd --zone=public --add-port=80:http --permanent 
	> 	# forewall-cmd --reload
	> 	#
	> 	# 重启 httpd 服务
	> 	# systemctl restart httpd
	
1. 主配置文件

	> 	#编辑主配置文件
	> 	vim /etc/httpd/conf/httpd.conf
	> 	
	> 	#服务器根目录
	> 	ServerRoot "/etc/httpd"
	> 	
	> 	#端口
	> 	Listen 80
	> 	
	> 	#域名+端口来标识服务器，没有域名用ip也可以
	> 	ServerName www.example.com:80
	> 	
	> 	#不许访问根目录
	> 	<Directory />
	> 	    AllowOverride none
	> 	    Require all denied
	> 	</Directory>
	> 	
	> 	# 文档目录
	> 	DocumentRoot "/var/www/html"
	> 	
	> 	# 对 /var/www目录访问限制
	> 	<Directory "/var/www">
	> 	    AllowOverride None
	> 	    # Allow open access:
	> 	    Require all granted
	> 	</Directory>
	> 	
	> 	# 对/var/www/html目录访问限制
	> 	<Directory "/var/www/html">
	> 	　　 Options Indexes FollowSymLinks
	> 	　　 AllowOverride None
	> 	 　　Require all granted
	> 	</Directory>
	> 	
	> 	# 默认编码 UTF-8
	> 	AddDefaultCharset UTF-8
	> 	
	> 	#EnableMMAP off
	> 	EnableSendfile on
	> 	
	> 	# include进来其它配置文件
	> 	IncludeOptional conf.d/*.conf

# 下载配置 mod_wsgi 连接 Django 使用#
1. 安装 httpd-devel

	> 	# yum install -y httpd-devel
1. 安装 mod_wsgi

	> 	# yum install mod_wsgi
1. 修改 httpd.conf

	> 	# vim /etc/httpd/conf/httpd.conf
	> 	# 
	> 	# 增加如下内容
	> 	# LoadModule  wsgi_module modules/mod_wsgi.so

