---
title: Nginx+Keepalived 高可用
date: 2022-03-23 15:13:47
categories:
- 运维
tags:
- Nginx
---



# Nginx+Keepalived 高可用

### 1、什么是Keepalived？

```
keepalived是为管理管理中保证能够提供高可用的一个服务软件，其功能称为心跳，可以防止单点故障。Copy
```

### 2、Keepalived安装

**每个Nginx下都要安装Keepalived**

```
1、下载地址：http://www.keepalived.org/download.html
2、cd /usr/local
3、tar -zxvf keepalived-1.2.18.tar.gz 
4、cd keepalived-1.2.18
5、安装软件包 yum install -y openssl openssl-devel
6、./configure --prefix=/usr/local/keepalived
7、make && make installCopy
```

### 3、Keepalived安装成Linux系统服务

```
1、将keepalived安装成Linux系统服务，因为没有使用keepalived的默认安装路径（默认路径：/usr/local）,安装完成之后，需要做一些修改工作
2、首先创建文件夹，将keepalived配置文件进行复制：
3、mkdir /etc/keepalived
4、cd /usr/local/keepalived-1.2.18
	cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
  然后复制keepalived脚本文件：
  cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
  cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
  ln -s /usr/local/sbin/keepalived /usr/sbin/
  ln -s /usr/local/keepalived/sbin/keepalived /sbin/
  可以设置开机启动：chkconfig keepalived onCopy
```

### 4、修改配置文件

```
1、cd /etc/keepalived/
2、修改keeplived.conf 配置高可用文件
3、添加nginx_check.sh脚本(重启Nginx脚本)Copy
! Configuration File for keepalived

vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_check.sh" #运行脚本，脚本内容下面有，就是起到一个nginx宕机以后，自动开启服务
    interval 2 #检测时间间隔
    weight -20 #如果条件成立的话，则权重 -20
}
# 定义虚拟路由，VI_1 为虚拟路由的标示符，自己定义名称
vrrp_instance VI_1 {
    state MASTER #来决定主从   BACKUP是从
    interface eth0 # 绑定虚拟 IP 的网络接口，根据自己的机器填写 指令ip a 查看
    virtual_router_id 121 # 虚拟路由的 ID 号， 两个节点设置必须一样
    mcast_src_ip 47.93.223.237 #填写本机ip
    priority 100 # 节点优先级,主要比从节点优先级高
    nopreempt # 优先级高的设置 nopreempt 解决异常恢复后再次抢占的问题
    advert_int 1 # 组播信息发送间隔，两个节点设置必须一样，默认 1s
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 将 track_script 块加入 instance 配置块
    track_script {
        chk_nginx #执行 Nginx 监控的服务
    }

    virtual_ipaddress {
        192.168.110.110 # 虚拟ip,也就是解决写死程序的ip怎么能切换的ip,也可扩展，用途广泛。可配置多个。
    }
}
Copy
#!/bin/bash
A=`ps -C nginx ¨Cno-header |wc -l`
if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    sleep 2
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fiCopy
```

### 5、常用指令

```
service keepalived start # 启动
service keepalived stop  # 关闭Copy
```

流程图

[![img](https://lukaixin0527.github.io/images/Keepalived流程图.jpg)](https://lukaixin0527.github.io/images/Keepalived流程图.jpg)

### 6、常见问题

```
1、如果nginx服务器宕机后，Keepalived没有重启ngxin，有可能是脚本没有权限重启
进入文件夹后，chmod 777 nginx_check.sh
```