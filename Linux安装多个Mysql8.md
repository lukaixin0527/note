---
title: Linux安装多个Mysql8
date: 2022-03-23 15:13:47
categories:
- 运维
tags:
- mysql
---

**需求：一台Centos服务器上离线安装多个Mysql8实例**

![image-20221104162247478](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221104162247478.png)

# **一、卸载MariaDB**

```
在CentOS中默认安装有MariaDB，是MySQL的一个分支，主要由开源社区维护。
CentOS 7及以上版本已经不再使用MySQL数据库，而是使用MariaDB数据库。
如果直接安装MySQL，会和MariaDB的文件冲突。
因此，需要先卸载自带的MariaDB，再安装MySQL。
```

## **1.1 查看版本**

```
rpm -qa|grep mariadb
```

## **1.2 卸载**

```
rpm -e --nodeps 文件名
```

## **1.3 检查是否卸载干净**

```
rpm -qa|grep mariadb
```

# **二、安装MySQL**

## **2.1 下载资源包**

MySQL官方地址:  https://dev.mysql.com/downloads/mysql/

![image-20221104163246765](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221104163246765.png)

## **2.2 解压**

```shell
.tar.gz后缀：tar -zxvf 文件名
.tar.xz后缀：tar -Jxvf 文件名

tar -Jxvf mysql-8.0.31-linux-glibc2.12-x86_64.tar.xz 

解压多次到 /usr/local/mysql目录下的mysql-8.0.31-3306、mysql-8.0.31-3307、mysql-8.0.31-3308、mysql-8.0.31-3309
```

![image-20221104163817815](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221104163817815.png)

# **三、用户和用户组**

## **3.1 创建用户组和用户**

```
# 创建一个用户组：mysql
groupadd mysql
# 创建一个系统用户：mysql，指定用户组为mysql
useradd -r -g mysql mysql
```

**创建用户组：**groupadd

**创建用户：**useradd

-r**：创建系统用户**

-g**：指定用户组**

## **3.2 数据目录**

**1、创建目录**

```
mkdir -p /usr/local/mysql/mysql-8.0.31-3306/data
mkdir -p /usr/local/mysql/mysql-8.0.31-3307/data
mkdir -p /usr/local/mysql/mysql-8.0.31-3308/data
mkdir -p /usr/local/mysql/mysql-8.0.31-3309/data
```

**2、赋予权限**

```
# 挨个赋予权限
# 更改属主和数组
chown -R mysql:mysql /usr/local/mysql/mysql-8.0.31-3306/data
# 更改模式
chmod -R 750 /usr/local/mysql/mysql-8.0.31-3306/data
```

# **四、初始化MySQL**

## **4.1 配置参数**

在/usr/local/mysql/mysql-8.0.31-3306/下，创建my.cnf配置文件，用于初始化MySQL数据库

```
[mysqld]
port=3306
user= mysql
socket= /usr/local/mysql/mysql-8.0.31-3306/data/mysql.sock
# 安装目录
basedir= /usr/local/mysql/mysql-8.0.31-3306
# 数据存放目录
datadir= /usr/local/mysql/mysql-8.0.31-3306/data
log-bin= /usr/local/mysql/mysql-8.0.31-3306/data/mysql-bin
innodb_data_home_dir=/usr/local/mysql/mysql-8.0.31-3306/data
innodb_log_group_home_dir=/usr/local/mysql/mysql-8.0.31-3306/data
#日志及进程数据的存放目录
log-error=/usr/local/mysql/mysql-8.0.31-3306/data/mysql.log
pid-file=/usr/local/mysql/mysql-8.0.31-3306/data/mysql.pid
# 服务端使用的字符集默认为8比特编码
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
character-set-server=utf8mb4
lower_case_table_names=1
autocommit=1
##################以上要修改的########################
skip-external-locking
key_buffer_size=256M
max_allowed_packet=1M
table_open_cache=1024
sort_buffer_size=4M
net_buffer_length=8K
read_buffer_size=4M
read_rnd_buffer_size=512K
myisam_sort_buffer_size=64M
thread_cache_size=128
#query_cache_size = 128M
tmp_table_size=128M
explicit_defaults_for_timestamp=true
max_connections=500
max_connect_errors=100
open_files_limit=65535


# 创建新表时将使用的默认存储引擎
default_storage_engine=InnoDB
innodb_data_file_path=ibdata1:10M:autoextend
innodb_buffer_pool_size=1024M
innodb_log_buffer_size=8M
innodb_flush_log_at_trx_commit=1
innodb_lock_wait_timeout=50
transaction-isolation=READ-COMMITTED
[mysqldump]
quick
max_allowed_packet=16M
[myisamchk]
key_buffer_size=256M
sort_buffer_size=4M
read_buffer=2M
write_buffer=2M
[mysqlhotcopy]
interactive-timeout

#开启bin log 功能
server-id=523306
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-ignore-db=sys
binlog-ignore-db=performance_schema
binlog_format=mixed
# 保留10天数据
binlog_expire_logs_seconds=864000

```

## **4.2 初始化**

进入bin目录

```
./mysqld --defaults-file=/usr/local/mysql/mysql-8.0.31-3306/my.cnf --basedir=/usr/local/mysql/mysql-8.0.31-3306 --datadir=/usr/local/mysql/mysql-8.0.31-3306/data --user=mysql --initialize
```

初始化成功后，会在`/usr/local/mysql/mysql-8.0.31-3306/data/mysql.log`下显示初始化密码

![image-20221104164436934](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221104164436934.png)

**参数（重要）**

defaults-file：指定配置文件（要放在–initialize 前面）

user： 指定用户

basedir：指定安装目录

datadir：指定初始化数据目录

intialize：初始化密码

# **五、启动MySQL**

## **5.1 启动**

进入bin目录

```
./mysqld_safe --defaults-file=/usr/local/mysql/mysql-8.0.31-3306/my.cnf &
```

**查看是否启动**

```
ps -ef|grep mysql
```

## **5.2 登录**

```
./mysql -uroot -p -h 127.0.0.1 --socket=../data/mysql.sock --port=3306
```

## **5.3 修改密码**

```
# 修改密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
# 刷新权限
FLUSH PRIVILEGES;
```

## **5.4 设置允许远程登录**

```
mysql> use mysql
mysql> update user set user.Host='%'where user.User='root';
mysql> flush privileges;
mysql> quit
```

到此3306服务就启动成功了，然后重复四、五步骤，分别启动3307、3308、3309即可。

