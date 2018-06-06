最近在vps上部署了 LAMP 环境，使用 Wordpress 搭建了这个博客。 在过程中遇到了一些问题，在这里做一下总结。

## 一. 准备工作

### 1. 系统版本：

>CentOS 7.3

### 2. 软件版本：

>1.Apache 2.4.6
>2.MySql 8.0.11
>3.PHP 7.2.6

### 3. 需要软件：
```
yum install -y wget
// 安装wget
```
>注：wget默认下载目录是当前执行命令的目录，为了便于管理下载文件可以新建一个文件夹专门用来保存下载的文件

## 二. Apache的安装

使用yum安装apache。
```
yum install -y httpd
systemctl enable httpd
// 将httpd设置为启动项
systemctl start httpd
// 启动apache服务
```
安装完成后，就可以在浏览器中输入你的ip地址，来检验是否安装成功。

## 三. Mysql的安装

这里介绍了两种安装mysql的方式，喜欢折腾的可以使用第一种rpm的安装方式，追求简单快捷的可以使用第二种yum的安装方式。

> rpm安装的mysql是8.0.11的版本， yum安装的是5.x.x的版本。

### 1. rpm安装

由于centos7自带mariadb所以需要先卸载mariadb。
```
rpm -qa mariadb*
// 查找所有被安装的mariadb相关的包
rpm -e xxx --nodeps
// 强制删除mariadb
```
安装相关依赖库
```
yum install -y net-tools* perl libaio
// 安装net-tools库,perl和libaio(这三个包在安装mysql-community-server时会需要用到)
```
下载Mysql的rpm bundle并解包
```
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.11-1.el7.x86_64.rpm-bundle.tar
// 下载mysql8.0.11的rpm包
tar xvf mysql-8.0.11-1.el7.x86_64.rpm-bundle.tar
// 解rpm包
```
添加用户和用户组
```
groupadd mysql
useradd -g mysql mysql
```
依次安装common, libs, client 和 server
```
rpm -ivh mysql-community-common-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.11-1.el7.x86_64.rpm
```
初始化mysql并进行相关配置
```
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
```
> 注：2018-06-04T09:51:05.698007Z 5 Note [Server] A temporary password is generated for root@localhost: l&feO>/Hn2Wy 其中 l&feO>/Hn2Wy是mysql的密码。

### 2. yum安装

rpm安装mysql的步骤较多，如果追求快捷的话可以使用yum安装mysql。
```
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
```
>yum默认的关于mysql的源比较老，只有5.x.x的版本。如果对版本要求较高，可以更换yum源。 如何使用yum安装mysql 8.x.x版本可以自行google或者百度。

## 四. PHP的安装

php的安装采用编译安装的方式。

安装所需的包

```
yum -y install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
// 安装相关依赖环境
yum -y install gcc gcc-c++
// 安装编译环境
yum -y install httpd-devel
// 安装httpd服务扩展工具, 没有安装httpd的需要安装httpd
```
下载php安装包并解压缩
```
wget http://php.net/distributions/php-7.2.6.tar.gz
// 下载php安装包
tar zxvf php-7.2.6.tar.gz
// 解压缩php安装包
cd php-7.2.6
// 进入解压缩后的php文件夹
```
配置安装相关的选项并安装
```
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
// 编译（这一步骤可能耗时较长）
make install
// 安装
```

> 编译过程中可能会出现 virtual memory exhausted: Cannot allocate memory 这个错误，这是因为内存不足造成的。可以通过swap来扩展内存解决。
> 如何使用可以查看链接: https://blog.csdn.net/taiyang1987912/article/details/41695895

配置相关环境变量
```
vim /etc/profile
// 在最下方加入
PATH=$PATH:/usr/local/php7/bin
export PATH
// path是你前面进行配置php安装包时的--prefix设置的目录下的bin目录
source /etc/profile
// 使配置立即生效
php -v
// 出现版本号就说明安装成功啦
```
配置php-fpm，并启动
```
cp php.ini-production /etc/php.ini
cp /usr/local/php7/etc/php-fpm.conf.default /usr/local/php7/etc/php-fpm.conf
cp /usr/local/php7/etc/php-fpm.d/www.conf.default /usr/local/php7/etc/php-fpm.d/www.conf
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm

service php-fpm start
//启动php-fpm
```
修改apache的配置文件
```
vim /etc/httpd/conf/httpd.conf
// 在AddType application后面加入下面一行
AddType application/x-httpd-php .php .phtml
// 在DirectoryIndex index.html加上index.php
DirectoryIndex index.php index.html
// 需要保证 httpd.conf 包含以下内容，如果没有请添加
LoadModule php7_module /usr/lib64/httpd/modules/libphp7.so

service httpd restart
// 重启apache
```
## 五. 安装phpmyadmin(可选)
>phpMyAdmin版本 4.8.1

[phpmyadmin](https://zh.wikipedia.org/wiki/PhpMyAdmin) 是一个浏览器端的数据库可视化管理工具，可以让你方便的对数据库进行管理。

```
wget https://files.phpmyadmin.net/phpMyAdmin/4.8.1/phpMyAdmin-4.8.1-all-languages.tar.gz
// 下载phpmyadmin的压缩包
tar zxvf phpMyAdmin-4.8.1-all-languages.tar.gz
// 解压phpmyadmin
cp -r phpMyAdmin-4.8.1-all-languages /var/www/html/phpmyadmin
// 拷贝phpmyadmin到网站根目录
cd /var/www/html/phpmyadmin
// 进入phpmyadmin
cp config.sample.inc.php config.inc.php
// 拷贝配置文件
vim config.inc.php
// 修改配置项
// $cfg['Servers'][$i]['host'] = 'localhost' 中的 localhost 修改为 127.0.0.1

service httpd restart
// 重启apache
```
然后就可以到浏览器中输入phpmyadmin的地址， 然后输入数据库的账号和密码就能查看数据库了。
>注：如果安装的MySql版本为8.x.x的版本，登录时会报错
>mysqli_real_connect(): The server requested authentication method unknown to the client [caching_sha2_password]
>mysqli_real_connect(): (HY000/2054): The server requested authentication method unknown to the client
>这是因为MySql在8.0的版本设置用户密码的默认认证方式改为了caching_sha2_password。可以通过修改用户密码的加密规则来解决。
>具体解决方案如下：
>```
>// 直接修改root密码
>ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'passw0rd';
>// 或
>// 新建用户，然后将赋予权限给新的用户
> CREATE USER username@localhost IDENTIFIED WITH mysql_native_password BY 'passw0rd';
> GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost';
>```
>然后就能正常登录啦。
## 六. WordPress的安装
> WordPress版本 4.9.4

在安装wordpress之前我们先在mysql中新建一个数据库，用于存放wordpress的数据。
```sql
mysql -u root -p
// 登录mysql
create database 'database_name';
exit;
```
如果你安装了phpmyadmin的话可以登录进phpmyadmin选择新建数据库，输入相关信息即可完成。
```
wget https://cn.wordpress.org/wordpress-4.9.4-zh_CN.tar.gz
// 下载 wordpress
tar zxvf wordpress-4.9.4-zh_CN.tar.gz
// 解压缩 wordpress
cp -r wordpress /var/www/html/
// 复制 wordpress 到网站根目录
cd /var/www/html/wordpress
// 进入wordpress目录
cp wp-config-sample.php wp-config.php
// 拷贝wordpress配置文件
```
接下来需要对wordpress进行一些配置
```
vim wp-config.php
```
```
define('DB_NAME', 'database_name_here');
// 将 database_name_here 修改为你之前新建的数据库的名称
define('DB_USER', 'username_here');
// 将username_here 修改为你的mysql用户名
define('DB_PASSWORD', 'password_here');
// 将password_here 修改为对应的密码
define('DB_HOST', 'localhost');
// 将 localhost 修改为 127.0.0.1
```
生成随机key （用于提高网站安全性，也可以不进行配置）
wordpress提供了一个自动生成随机key的地址：https://api.wordpress.org/secret-key/
用生成的key替换下方的内容
```
define('AUTH_KEY', 'put your unique phrase here');
define('SECURE_AUTH_KEY', 'put your unique phrase here');
define('LOGGED_IN_KEY', 'put your unique phrase here');
define('NONCE_KEY', 'put your unique phrase here');
define('AUTH_SALT', 'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT', 'put your unique phrase here');
define('NONCE_SALT', 'put your unique phrase here');
```
这样就完成了wordpress的安装。
> 注: 如果wordpress安装更新需要ftp用户名和密码可以在 wp-config.php 最后一句加上
> ```
> define('FS_METHOD', "direct");
> ```
> 然后在shell中输入命令修改一下目录的权限就行
> ```
> chmod 777 -R /var/www/html/wordpress/
> // 将当前目录下的所有文件及子目录的文件拥有者权限设置为读、写、可执行，文件拥有者所在的用户组成员具备读、写、可执行权限，其它用户也具备读、写、可执行权限
> ```

最后安装完成的效果
[caption id="attachment_12" align="alignnone" width="648"]<a href="http://ihikaru.com/wordpress/wp-content/uploads/2018/06/index-firshot-1.png"><img src="http://ihikaru.com/wordpress/wp-content/uploads/2018/06/index-firshot-1-1024x524.png" alt="wordpress主页" width="648" height="332" class="size-large wp-image-12"></a> 安装完成后的wordpress主页[/caption]