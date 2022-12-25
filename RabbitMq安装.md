---
title: RabbitMq安装
date: 2022-03-23 15:13:47
categories:
- 运维
tags:
- 消息队列
- RabbitMq
---



## 1、简介

**官网**：https://www.rabbitmq.com/

[RabbitMQ](https://so.csdn.net/so/search?q=RabbitMQ&spm=1001.2101.3001.7020)是一个开源的遵循`AMQP`协议实现的基于[Erlang](https://so.csdn.net/so/search?q=Erlang&spm=1001.2101.3001.7020)语言编写，支持多种客户端（语言），用于在分布式系统中存储消息，转发消息，具有高可用高可扩性，易用性等特征。
![image-20210306004740486](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img3c01cd2a894605d1e77f665dea4ba39a.png)

------

## 2、下载安装启动RabbitMQ

**环境准备**：阿里云[centos7](https://so.csdn.net/so/search?q=centos7&spm=1001.2101.3001.7020).6 服务器

```
# 查看系统版本
[root@zsr ~]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.6.1810 (Core) 
Release:	7.6.1810
Codename:	Core
```

### 2.1、下载RabbitMQ

**下载地址**：https://www.rabbitmq.com/download.html
![image-20210505195722340](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgc2a43d7b156037d026377e58f000c0a1.png)
选择对应的系统版本点击下载，下载后会得到`.rpm`文件
![image-20210306171625359](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgd30ba3494143e490586b644a777d1a86.png)

### 2.2、下载Erlang

> RabbitMQ是采用 Erlang语言开发的，所以系统环境必须提供 Erlang环境，需要是安装 Erlang

> `Erlang`和`RabbitMQ`版本对照：https://www.rabbitmq.com/which-erlang.html
>
> ![image-20210306105403265](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img982bab51c7a7c5e7859aac80b73246e8.png)

这里安装最新版本3.8.14的`RabbitMQ`，对应的`Erlang`版本推荐`23.x`，我们下载`erlang-23.2.7-2.el7.x86_64.rpm`

下载地址：https://packagecloud.io/rabbitmq/erlang/packages/el/7/erlang-23.2.7-2.el7.x86_64.rpm
![image-20210306155749846](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img434d500793ba3b7102502060a2d24dd5.png)
其中的`el7`表示Red Hat 7.x，即`CentOS 7.x`

点击右上角下载即可得到`.rpm`文件
![image-20210306171649898](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img1f7e491967213fae693b324a23e1404e.png)

### 2.3、安装Erlang

首先将下载好的文件上传到服务器，创建一个文件夹用来存放文件

```
[root@zsr ~]# mkdir -p /usr/rabbitmq
```

再利用`xftp`工具将上述下载的两个`.rpm`文件上传到服务器的刚创建的文件夹中
![image-20210306171754048](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img329ae85431345deba319246cf34beced.png)
然后切换到`/usr/rabbitmq`目录，解压安装`erlang`

```
# 解压
rpm -Uvh erlang-23.2.7-2.el7.x86_64.rpm

# 安装
yum install -y erlang

123456
```

![image-20210306172313930](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgbc258420e09d877321b8d84bac8dcbf5.png)
安装完成后输入如下指令查看版本号

```
erl -v
```

![image-20210306172327030](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgc06df0aa6fa2e3042c7dc816bff2c6a4.png)

### 2.4、安装RabbitMQ

在`RabiitMQ`安装过程中需要依赖`socat`插件，首先安装该插件

```
yum install -y socat
```

然后解压安装`RabbitMQ`的安装包

```
# 解压
rpm -Uvh rabbitmq-server-3.8.14-1.el7.noarch.rpm

# 安装
yum install -y rabbitmq-server
```

### 2.5、启动RabbitMQ服务

```
# 启动rabbitmq
systemctl start rabbitmq-server

# 查看rabbitmq状态
systemctl status rabbitmq-server
```

显示`active`则表示服务安装并启动成功
![image-20210306173012889](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img8670d8cd1b59902bfdba68d9925ee92c.png)
其他命令：

```
# 设置rabbitmq服务开机自启动
systemctl enable rabbitmq-server

# 关闭rabbitmq服务
systemctl stop rabbitmq-server

# 重启rabbitmq服务
systemctl restart rabbitmq-server
```

------

## 3、RabbitMQWeb管理界面及授权操作

### 3.1、安装启动RabbitMQWeb管理界面

> 默认情况下，rabbitmq没有安装web端的客户端软件，需要安装才可以生效

```
# 打开RabbitMQWeb管理界面插件
rabbitmq-plugins enable rabbitmq_management
```

![image-20210306191329997](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgfc1fd4d55686496ae5cbe96675c21482.png)
然后我们打开浏览器，访问`服务器公网ip:15672`（注意打开阿里云安全组以及防火墙的15672端口），就可以看到管理界面
![image-20210306193911485](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imga48b7fdfa923e62a803ab273a2228a1d.png)

`rabbitmq`有一个默认的账号密码`guest`，但该情况仅限于本机localhost进行访问，所以需要添加一个远程登录的用户

### 3.2、添加远程用户

**RabbitMq角色**

RabbitMQ中的角色分为如下五类：none、management、policymaker、monitoring、administrator

(1) 超级管理员(administrator)可登陆管理控制台(启用management plugin的情况下)，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

(2) 监控者(monitoring)可登陆管理控制台(启用management plugin的情况下)，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)(3) 策略制定者(policymaker)可登陆管理控制台(启用management plugin的情况下), 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。

(4) 普通管理者(management)仅可登陆管理控制台(启用management plugin的情况下)，无法看到节点信息，也无法对策略进行管理。

(5) 其他(none)无法登陆管理控制台，通常就是普通的生产者和消费者。

**角色权限指令**

​    **1、查询用户列表** ：   `rabbitmqctl list_users`

​    **2、新建用户**： `rabbitmqctl add_user user password`  

例如创建用户test01密码test01：`rabbitmqctl add_user test01 test01`    

​	会提示：完毕。 不要忘记授予用户对某些虚拟主机的权限！

![Image](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgImage.png)

**3、设置用户角色**：`rabbitmqctl set_user_tags User Tag`  

例如设置用户test01为none权限：`rabbitmqctl set_user_tags test01 none`

![image-20221101095119540](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221101095119540.png)

**4、设置用户权限**：`rabbitmqctl set_permissions [-p vhostpath] {user} {conf} {write} {read}`         

（1）-p vhostpath ：用于指定一个资源的命名空间，例如 –p / 表示根路径命名空间；       

（2）user：用于指定要为哪个用户授权填写用户名；       

（3）conf:一个正则表达式match哪些配置资源能够被该用户配置；       

（4）write:一个正则表达式match哪些配置资源能够被该用户读；       

（5）read:一个正则表达式match哪些配置资源能够被该用户访问。      

 例如：用于设置test01用户拥有对所有资源的 读写配置权限       `rabbitmqctl set_permissions -p / test01 ".*" ".*" ".*"`

**5、查询指定用户的权限信息** `rabbitmqctl list_user_permissions User`

 例如查询test01用户权限 ：`rabbitmqctl list_user_permissions test01`

![image-20221101095232977](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221101095232977.png)

**6、删除用户** rabbitmqctl delete_user User 

例如删除test01用户 ： `rabbitmqctl delete_user test01`