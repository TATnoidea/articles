最近在vps上部署了lamp环境，使用wordpress搭建了这个博客。 在过程中遇到了一些问题，在这里做一下总结。

 

\###一. 准备工作

 

\####1. 系统版本：

 

CentOS 7.3

 

\####2. 软件版本：

 

\1. Apache 2.4.6

\2. MySql 

\3. PHP

 

\####3. 需要软件：

\```

yum install -y wget

// 安装wget

\```

注：wget默认下载目录是当前执行命令的目录，为了便于管理下载文件可以新建一个文件夹专门用来保存下载的文件

 

\###二. Apache的安装

 

使用yum安装apache。

\```

yum install -y httpd

systemctl enable httpd

// 将httpd设置为启动项

systemctl start httpd

// 启动apache服务

\```

安装完成后，就可以在浏览器中输入你的ip地址，来检验是否安装成功。

 

 

 

\###三. Mysql的安装

 

这里介绍了两种安装mysql的方式，喜欢折腾的可以使用第一种rpm的安装方式，追求简单快捷的可以使用第二种yum的安装方式。

 

\> rpm安装的mysql是8.0.11的版本， yum安装的是5.x.x的版本。

 

\####1. rpm安装

 

由于centos7自带mariadb所以需要先卸载mariadb。

\```

rpm -qa mariadb*

// 查找所有被安装的mariadb相关的包

rpm -e xxx --nodeps

// 强制删除mariadb

\```

使用rpm安装 mysql 8.0.11。

\```

yum install -y net-tools* perl libaio

// 安装net-tools库,perl和libaio(这三个包在安装mysql-community-server时会需要用到)

wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.11-1.el7.x86_64.rpm-bundle.tar

// 下载mysql8.0.11的rpm包

tar xvf mysql-8.0.11-1.el7.x86_64.rpm-bundle.tar

// 解rpm包

​    

groupadd mysql

useradd -g mysql mysql

// 添加用户组与用户

​    

rpm -ivh mysql-community-common-8.0.11-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-8.0.11-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-8.0.11-1.el7.x86_64.rpm

rpm -ivh mysql-community-server-8.0.11-1.el7.x86_64.rpm

// 依次安装

​    

mysqld --initialize --user=mysql

// 初始化mysql

systemctl enable mysqld

// 将mysql设置为启动项

systemctl start mysqld

// 启动mysql

vim /var/log/mysqld.log

// 查看初始化的密码，具体说明请看下方注解

mysql -u root -p

//登录mysql

ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';

// 修改密码

flush privileges;

// 刷新权限列表

exit

//退出mysql，mysql安装完成

\```

\> 注：2018-06-04T09:51:05.698007Z 5 Note [Server] A temporary password is generated for root@localhost: l&feO>/Hn2Wy 其中 l&feO>/Hn2Wy是mysql的密码。

 

\####2. yum安装

 

rpm安装mysql的步骤较多，如果追求快捷的话可以使用yum安装mysql。

\```

yum install -y mariadb mariadb-server

// 安装mariadb

systemctl enable mariadb

// 设置mariadb为启动项

systemctl start mariadb

// 启动mariadb

mysql_secure_installation

// mariadb的相关简单配置

// Enter current password for root (enter for none):<–初次运行直接回车

// Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车

// New password: <– 设置root用户的密码

// Re-enter new password: <– 再输入一次你设置的密码

// Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车

// Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车,

// Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车

// Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车

mysql -u root -p

// 登录测试，登录成功安装完成

\```

yum默认的关于mysql的源比较老，只有5.x.x的版本。如果对版本要求较高，可以更换yum源。

如何使用yum安装mysql 8.x.x版本可以自行google或者百度，也可以参考文章结尾的相关链接。

 

\###四. PHP的安装

php的安装采用编译安装的方式。

\```

yum -y install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel

// 安装相关依赖环境

yum -y install gcc gcc-c++

// 安装编译环境

yum -y install httpd-devel

// 安装httpd服务扩展工具, 没有安装httpd的需要安装httpd

 

wget http://php.net/distributions/php-7.2.6.tar.gz

// 下载php安装包

tar zxvf php-7.2.6.tar.gz

// 解压缩php安装包

cd php-7.2.6

// 进入解压缩后的php文件夹

./configure --prefix=/usr/local/php7 \

 --with-apxs2=/usr/bin/apxs \

 --with-curl \

 --with-freetype-dir \

 --with-gd \

 --with-gettext \

 --with-iconv-dir \

 --with-kerberos \

 --with-libdir=lib64 \

 --with-libxml-dir \

 --with-mysqli \

 --with-openssl \

 --with-pcre-regex \

 --with-pdo-mysql \

 --with-pdo-sqlite \

 --with-pear \

 --with-png-dir \

 --with-xmlrpc \

 --with-xsl \

 --with-zlib \

 --enable-fpm \

 --enable-bcmath \

 --enable-libxml \

 --enable-inline-optimization \

 --enable-gd-native-ttf \

 --enable-mbregex \

 --enable-mbstring \

 --enable-opcache \

 --enable-pcntl \

 --enable-shmop \

 --enable-soap \

 --enable-sockets \

 --enable-sysvsem \

 --enable-xml \

 --enable-zip

// 配置相关参数

// 其中有两个参数比较重要

// --prefix参数配置的是php的目标安装目录

// --with-apxs2是httpd-devel的安装目录，安装目录可以使用 find / -name apxs 进行查找

make

// 编译

make install

// 安装

\```

 

 