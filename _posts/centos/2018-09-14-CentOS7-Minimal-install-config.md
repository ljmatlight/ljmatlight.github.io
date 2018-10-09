---
layout: post
title:  "CentOS7 最小化安装配置"
categories: CentOS7
tags: CentOS
author: tengpeng.gao
---

* content
{:toc}


## 简述

- CentOS 7 最小化安装版本：CentOS-7-x86_64-Minimal-1708

## 基础配置

### 配置网络

- VM选择桥接
- 手工配置网络地址
- 验证可以访问外网

VM克隆系统 设置静态 IP

```bash

cd /etc/sysconfig/network-scripts/

vi ifcfg-eno16777736

```

注释掉 UUID， HWADDR

```conf

TYPE="Ethernet"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="yes"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
NAME="eno16777736"
#UUID="10f47dcb-cc95-4aad-a56c-36fe7920f431"
ONBOOT="yes"
IPADDR0="192.168.199.200"
PREFIX0="24"
GATEWAY0="192.168.199.1"
DNS1="8.8.8.8"
DNS2="9.9.9.9"
#HWADDR="00:0C:29:E3:95:59"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"

```

重启网络服务

```
service network restart
```

### 修改主机名
    
```bash
hostnamectl set-hostname serverHostName 
```

### 系统时间同步配置

```bash

yum install ntpdate

# 同步时间服务器
ntpdate time.nist.gov
# 或
ntpdate -u 0.pool.ntp.org

```

同步时间可能有问题，参见[解决CentOS7下用ntpdate同步时间问题](https://blog.csdn.net/qq_27754983/article/details/69386408)


## 安装基本工具

### 安装net-tools  
```bash
yum -y install net-tools  
```
### 安装 wget
```bash
yum -y install wget
```
### 安装 curl
```bash
yum -y install curl
```

## 基本命令

### 查找安装路径：
```bash
whereis nginx
```

### 查询nginx进程：

```bash
ps aux|grep nginx
```

### 查看 CentOS 内核版本：
```bash
uname -r
```

### 查看 gcc 是否安装

```bash
rpm -qa|grep gcc
```
### 卸载软件
    
需要看你的软件包格式：

```bash

# 如果你带有yum，可以直接
yum remove xxx
 
# 如果是rpm包，
rpm -e xxx

# tar包的话需要你直接删除该文件或者
make uninstall xxx

```

卸载 Docker:

```bash


# 查看
yum list installed | grep docker 

# 卸载
yum -y remove docker.xxx.x86_64

# 删除
rm -ef /var/lib/docker

```

```bash
#查看ip信息
ip add

#显示当前路径的全路径 
pwd

#文件复制 
cp -r /bashrc /bak/bashrc

#更新
yum update 

tail -f /data/logs/xxxx/xxxx.log 

#查看文档内容
cat    

#分页查看文档内容
more   

#列出所有文件
ls -a  

#拷贝文件夹及文件夹内文件
cp -r tomcat-xxxx tomcat-xxxx-new   

#强制删除文件夹或文件
rm -rf logs   

#清空文件内容
echo "">catalina.out   


# 找到 tomcat-x-cas-server 的进程，
# 第二个参数是 pid
# 通过 pid 杀死进程
ps -ef | grep "tomcat-x-cas-server" | grep -v grep | awk '{print $2}' | xargs kill -9

```
 
## 开发环境

### 安装 java

1.卸载 自带的 openjdk 

```bash
rpm -qa|grep java
```

rpm -e -nodeps java-xxx

2.从 Oracle 官网下载 jdk-8u181-linux-x64.tar.gz

3.解压

```bash
tar –xzvf jdk-8u45-linux-x64.gz
```

4.jdk的配置

```bash
vi /etc/profile
```

```bash
export JAVA_HOME=jdk的绝对路径
export PATH=$PATH:$JAVA_HOME/bin
```

5.测试安装是否成功
使用`reboot`命令重启系统使环境变量生效。

```bash
java -version
```

### 安装 Maven

```bash
yum -y install maven
```

### 安装 Git
```bash
yum -y install git
```

### 安装 tomcat
```bash
# 通过 wget 方式下载 apache-tomcat-8.5.23.tar.gz
wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.23/bin/apache-tomcat-8.5.23.tar.gz

# 解压 apache-tomcat-8.5.23.tar.gz
tar -xzvf apache-tomcat-8.5.23.tar.gz

# 启动 tomcat 
./startup.sh

# 将8080端口添加到防火墙例外并重启
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload
```

### 安装 MySQL
```bash
#1. 下载 mysql 的 repo 源
wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm
#2. 安装 mysql 的 repo 源
rpm -ivh mysql57-community-release-el7-8.noarch.rpm
#3. 安装 mysql
yum -y install mysql-server
```

Mysql5.7默认安装之后root是有密码的。

获取MySQL的临时密码
  为了加强安全性，MySQL5.7为root用户随机生成了一个密码，在error log中，关于error log的位置，如果安装的是RPM包，则默认是/var/log/mysqld.log。 
  只有启动过一次mysql才可以查看临时密码

```bash
#查看原始密码
grep 'temporary password' /var/log/mysqld.log

#将3306端口添加到防火墙例外并重启
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

```mysql

#修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'eFeG20125';

#授权远程网络访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.199.%' IDENTIFIED BY 'eFeG20125' WITH GRANT OPTION;
flush privileges;

```

 

### 安装 Redis
```bash
#1. 设置 Redis 的仓库地址
yum -y install epel-release
#2. 安装 Redis
yum -y install redis
#3. 配置 redis.conf
    #bind 127.0.0.1 
    requirepass redisPassword 
#4. 开放 redis 端口
# 将6379端口添加到防火墙例外并重启
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --reload

```

### 安装 Nginx

```bash

yum -y install nginx

# 重启 nginx 服务
service nginx restart

# 将 80 端口添加到防火墙例外并重启
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload

```
 