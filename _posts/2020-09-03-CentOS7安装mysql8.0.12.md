---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 学习记录
---
# 一、配置yum源
## 1.下载mysql源安装包
本次下载目录为：/home/目录，因此进入：cd /home

执行下载命令：

```
wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm

```
## 2.安装mysql源
下载完成后使用下面命令安装源：

```
yum localinstall mysql80-community-release-el7-1.noarch.rpm

```
## 3.检查是否安装成功

```
yum repolist enabled | grep "mysql.*-community.*"

```
## 4.修改安装版本（非必须）
如果需要安装指定版本的mysql，可以修改/etc/yum.repos.d/mysql-community.repo源，改变默认安装的mysql版本。

例如要安装5.7版本，将5.7源的enabled=0改成enabled=1，将8.0的enabled=1改成enabled=0即可，如下（本次未做修改，直接安装最新版8.0.12）：

![换版本](https://img-blog.csdn.net/2018100715082168?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NTkxNzU2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
# 二、安装mysql
直接使用命令：yum install mysql-community-server即可。

```
yum install mysql-community-server
# 三、启动mysql服务

```
## 1.启动

```
systemctl start mysqld
# 或者
service mysqld start

```
## 2.查看启动状态

```
systemctl status mysqld
# 或者
service mysqld status

```
![启动状态](https://img-blog.csdn.net/20181007150954915?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NTkxNzU2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 3.设置开机启动

```
systemctl enable mysqld
systemctl daemon-reload

```
# 四、配置及部分命令
## 1.修改登录密码
mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改：

```
grep 'temporary password' /var/log/mysqld.log

```
然后修改密码：

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'TestBicon@123';
# 或者
set password for 'root'@'localhost'=password('TestBicon@123');

```
### 修改密码策略

```
# 查看密码策略
show variables like '%validate_password.policy%';
show variables like '%validate_password.length%';
# 修改密码策略
set global validate_password.policy=0;  # 设置为弱口令
set global validate_password.length=1;  # 密码最小长度为1

```
## 2.添加远程登录用户

```
GRANT ALL ON *.* TO 'root'@'%';

# 如果报错：ERROR 1410 (42000): You are not allowed to create a user with GRANT
update user set host='%' where user ='root';

# 刷新权限
flush privileges;

```
3.sqlyog/navicate链接时出现2058异常

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
# 如果报错：ERROR 1396 (HY000): Operation ALTER USER failed for 'root'@'localhost'则使用下面命令：
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

```
# 五、彻底卸载mysql
## 1.卸载软件

```
yum remove mysql-community-server

```
### 完成后使用rpm -qa|grep mysql命令查看，如果有查询结果，则使用yum remove 名称清理掉。如图：
![](https://img-blog.csdn.net/20181007152057292?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NTkxNzU2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
### 再使用命令rpm -qa | grep -i mysql查看，如果有结果使用rpm -e 名称卸载。例如：
![](https://img-blog.csdn.net/20181007152108839?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NTkxNzU2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 2.删除文件

```
rm -rf /var/lib/mysql
rm /etc/my.cnf
rm -rf /usr/share/mysql-8.0

```
### 如果需要重新安装，在安装完成启动之前可以先对mysql目录赋予权限防止异常发生：

```
chmod -R 777 /var/lib/mysql

```