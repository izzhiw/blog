---
title: 在 CentOS 7 中安装 Joomla! 的教程
categories:
  - website-building
tags:
  - Joomla!
date: 2017-05-04 14:21:49
---
{% note primary %}

目标：
1. 在 CentOS-7-x86_64 系统中安装最新版本的 Joomla! CMS。
2. 支持 PHP 7。

{% endnote %}
{% note warning %}

条件：
1. CentOS 7 系统安装就绪。
2. 普通用户帐号，拥有 `sudo` 权限。

{% endnote %}
{% note success %}

步骤：
1. 安装 LAMP 之 A (Apache)。
2. 安装 LAMP 之 M (MariaDB)。
3. 安装 LAMP 之 P (PHP)。
4. 配置 MariaDB 数据库。
5. 配置 Joomla! 程序。
6. 配置防火墙。
7. 安装 Joomla! 程序。

{% endnote %}
<!-- more -->

### 安装 Web 服务
- 更新系统。
```
sudo yum update -y
```
- 安装 Apache。
```
sudo yum install -y httpd
```
- 启动服务，并配置开机即启动。
```
sudo systemctl start httpd
sudo systemctl enable httpd
```
### 安装数据库服务
- 向系统添加 MariaDB 仓库。
```
sudo vim /etc/yum.repos.d/mariadb.repo
```
  编辑内容如下。
```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```
- 更新仓库。
```
sudo yum update -y
```
- 安装 MariaDB。
```
sudo yum install -y mariadb-server
```
- 启动服务，并配置开机即启动。
```
sudo systemctl start mariadb
sudo systemctl enable mariadb
```
### 安装 PHP 及相关模块
- 安装并更新 PHP 7 仓库。
```
sudo yum install -y http://dl.iuscommunity.org/pub/ius/stable/CentOS/7/x86_64/ius-release-1.0-14.ius.centos7.noarch.rpm
sudo yum update -y
```
- 安装 PHP 7。
```
sudo yum -y install php70u php70u-pdo php70u-mysqlnd php70u-opcache php70u-xml php70u-mcrypt php70u-gd php70u-devel php70u-intl php70u-mbstring php70u-bcmath php70u-json php70u-iconv
```
### 配置 MariaDB 数据库
- 执行数据库安全配置脚本。
```
sudo mysql_secure_installation
```
  完成如下配置（中括号内的大写 `Y` 为默认选项，回车即可），为数据库 `root` 用户设置强密码（初始密码为空）。
```
Enter current password for root (enter for none):
Change the root password? [Y/n]
Remove anonymous users? [Y/n]
Disallow root login remotely? [Y/n]
Remove test database and access to it? [Y/n]
Reload privilege tables now? [Y/n]
```
- 为 Joomla! 新建一个数据库和一个数据库用户。
  以数据库 root 用户登录 MariaDB。
```
mysql -u root -p
```
  按提示输入密码，并新建 `joomladb` 数据库和 `joomlauser@localhost` 用户（注意修改单引号内 `password` 为新用户密码）。
```
MariaDB [(none)]>create database joomladb;
MariaDB [(none)]>create user joomlauser@localhost identified by 'password';
MariaDB [(none)]>grant all privileges on joomladb.* to joomlauser@localhost;
MariaDB [(none)]>flush privileges;
MariaDB [(none)]>exit
```
### 配置 Joomla! 程序
- 下载最新版本的 Joomla!，写作此教程时的最新版本号为 3.7.0。
```
wget https://github.com/joomla/joomla-cms/releases/download/3.7.0/Joomla_3.7.0-Stable-Full_Package.tar.gz
```
- 解压文件至 Web 服务的根目录 `/var/www/html/`。
```
sudo tar -xvzf Joomla_3.7.0-Stable-Full_Package.tar.gz -C /var/www/html/
```
- 配置 Web 服务根目录的权限。
```
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 775 /var/www/html/
```
- 编辑 Web 服务的配置文件。
```
sudo vim /etc/httpd/conf/httpd.conf
```
  查找 `/var/www/html/` 下的 `AllowOverride` 项，将 `AllowOverride None` 改为 `AllowOverride All`。
  保存文件，并重启 Web 服务。
```
sudo systemctl restart httpd
```
### 配置防火墙
- 查看系统启用的防火墙是 firewall 还是 iptables。
```
systemctl status firewalld
systemctl status iptables
```
  `Active:` 状态为 `active (exited)` 的程序是系统启用的防火墙。
- 若 firewall 启用，
```
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
- 若 iptables 启用，
```
sudo vim /etc/sysconfig/iptables
```
  在 `-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT` 行的下面，添加如下内容。
```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
```
  保存文件，并重启 iptables。
```
sudo systemctl restart iptables
```
### 安装 Joomla! 程序
- 打开浏览器，输入你的服务器地址，访问 Joomla! 安装向导。
- 特别注意，Database Type 请选择 MySQL(PDO)！
- 安装完成后，删除 installation 目录。
- 再次访问你的网站，地址后加 `/administrator` 进入管理登录页面。
- 开始你的 Joomla! 之旅。

(END)

> 本文原创，[转载请注明出处](http://blog.zhengzhiwei.net/2017/05/04/install-joomla-cms-on-centos-7/)
> 我是 zZhiw，[更多内容请关注](https://zhengzhiwei.net)

