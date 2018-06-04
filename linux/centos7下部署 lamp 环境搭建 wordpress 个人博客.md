最近在vps上部署了lamp环境，使用wordpress搭建了这个博客。 在过程中遇到了一些问题，在这里做一下总结。

### 一. 准备工作

#### 1. 系统版本：

CentOS 7.3

####2. 软件版本：

1. Apache 2.4.6
2. MySql 
3. PHP

#### 3. 需要软件：

```
yum install -y wget
// 安装wget
```

> 注：wget默认下载目录是当前执行命令的目录，为了便于管理下载文件可以新建一个文件夹专门用来保存下载的文件



### 二. Apache的安装

使用yum安装apache。

```
yum install -y httpd
systemctl enable httpd
// 将httpd设置为启动项
systemctl start httpd
// 启动apache服务
```

安装完成后，就可以在浏览器中输入你的ip地址，来检验是否安装成功。



###三. Mysql的安装

这里介绍了两种安装mysql的方式，喜欢折腾的可以使用第一种rpm的安装方式，追求简单快捷的可以使用第二种yum的安装方式。

> rpm安装的mysql是8.0.11的版本， yum安装的是5.x.x的版本。

#### 1. rpm安装

由于centos7自带mariadb所以需要先卸载mariadb。

```
rpm -qa mariadb*
// 查找所有被安装的mariadb相关的包

rpm -e xxx --nodeps
// 强制删除mariadb
```

使用rpm安装 mysql 8.0.11。

```
yum install -y net-tools* perl libaio
// 安装net-tools库,perl和libaio(这三个包在安装mysql-community-server时会需要用到)
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.11-1.el7.x86_64.rpm-bundle.tar
// 下载mysql8.0.11的rpm包
tar xvf mysql-8.0.11-1.el7.x86_64.rpm-bundle.tar
// 解rpm包

groupadd mysql
useradd -g mysql mysql
// 添加用户组与用户

rpm -ivh mysql-community-common-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.11-1.el7.x86_64.rpm
// 依次安装

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

> 注：2018-06-04T09:51:05.698007Z 5 [Note][MY-010454] [Server] A temporary password is generated for root@localhost: l&feO>/Hn2Wy 其中 l&feO>/Hn2Wy时mysql密码。

####2. yum安装

rpm安装mysql的步骤较多，如果追求快捷的话可以使用yum安装mysql。

```
yum install -y mariadb
```



> yum默认的关于mysql的源比较老，只有5.x.x的版本。如果对版本要求较高，可以更换yum源。



### 四. PHP的安装

