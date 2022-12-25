---
title: Docker容器
date: 2022-03-23 15:13:47
categories:
- 运维
tags:
- Docker
---

# Docker容器

最近因为工作需要，利用课余时间通过视频、书籍学习Docker，并在此记录笔记。

# 第一章Docker简介

## 1.1 Docker能解决什么问题

```
问题：开发人员经常说："它在我的机器可以正常运行"，言下之意就是，其他机器很可能跑不了
解决：开发人员可以使用 Docker 来解决 "它在我的机器可以正常运行" 的问题，它会将运行程序的相关配置打包（打包成
     一个镜像），然后直接搬移到新的机器上运行。Copy
```

## 1.2 什么是虚拟机技术

```
概述：它可以在一种操作系统里面运行另一种操作系统，比如 VMware workstation 虚拟化产品提供了虚拟的硬件，在
Windows 系统里面运行 Linux 系统。
问题：
	1、占用资源多,每个虚拟机会独占一部分内存和硬盘空间
	2、冗余步骤多,虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。
	3、启动慢,启动硬件上的操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，虚拟机才能真正运行Copy
```

## 1.3 什么是容器

```
1、容器技术
		由于虚拟机存在的这些缺点，发展处了另一种虚拟化技术：容器
		容器与虚拟机有所不同：
    【虚拟机】通过虚拟软件中间层将一台或者多台独立的虚拟机器运行在物理硬件之上。
    (比如使用 VMware workstation 软件生成虚拟机在电脑上)
    【容器】则是直接运行在操作系统内核之上。(容器直接安装后就可以运行在电脑上)，是进程级别的，
     并对进程进行了隔离，而不是模拟一个完整的操作系统,是轻量级别的Copy
```



**如上图所示，鲸鱼就是docker，集装箱就是各个容器，海洋就是宿主机，比如Liunx**

```
2、容器与虚拟机比较
  容器是进程级别的，相比虚拟机有很多优势
  容器：体积小、启动快、资源占用少(只占用需要的资源)Copy
```



## 1.4 Docker内部结构

```
1、Docker镜像(image)
镜像是Docker中的一个模板。通过Docker镜像 来创建Docker容器，一个镜像可以创建多个容器。
镜像好比一个类，容器则为一个一个的对象。
  
2、容器(container)
容器是基于镜像创建的运行实例，一个容器中可以运行一个或者多个应用程序。
可以理解为，一个精简版的Liunx环境+要运行的应用程序
  
3、Docker仓库(repository)
仓库是集中存放镜像文件的场所。
仓库(Repository)和仓库注册服务器(Registry)有所不同，仓库注册服务器上存放多个仓库，每个仓库又包含了多个镜像，每个镜像有不同的标签(tag)。
  Docker仓库分为公有仓库和私有仓库，私有仓库将镜像存入私有服务器即可。Copy
```

# 第二章Docker安装

## 2.1、ScentOS安装Docker安装卸载与启停

安装步骤Docker官网有详细说明 **https://docs.docker.com/engine/install/centos/**

```
1、卸载旧版本
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
2、安装软件包，提供yum-config-manager实用程序
sudo yum install -y yum-utils
3、设置镜像
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
4、安装最新的Docker
sudo yum install docker-ce docker-ce-cli containerd.io
5、启动Docker
sudo systemctl start docker
6、测试Docker
sudo docker run hello-worldCopy
```

## 2.2、常用指令

- 启动Docker：systemctl start docker
- 停止docker：systemctl stop docker
- 重启docker：systemctl restart docker
- 查看docker状态： systemctl status docker
- 开机自动启动docker： systemctl enable docker
- Docker帮助指令：docker –help

# 第三章、Docker的镜像操作

## 3.1、什么是Docker镜像

```
docker镜像是由文件系统叠加而成,是一种文件的存储形式。最低端是一个问价引导系统，即bootfs
这
很像典型的Linux/Unix的引导文件系统。Docker用户几乎永远不会和引导系统有什么交互。实际上，当一个
容器启动后，它将会被移动到内存中，而引导文件系统则会被卸载，以留出更多的内存供磁盘镜像使用。
Docker容器启动是需要的一些文件，而这些文件就可以称为Docker镜像Copy
```

## 3.2、常用指令

- 列出docker下的已安装所有镜像：docker images

- 只显示镜像ID：docker images -q

- 搜索镜像：docker search[OPTIONS] 镜像名称

  ```
              OPTIONS: -s 100 列出关注数大于指定值的镜像
                       --no-trunc 显示完整的镜像描述Copy
  ```

- 删除镜像：docker rmi 镜像ID (rmi —— remove images的缩写)

- 删除所有镜像：docker rmi `docker images -q`

## 3.3、拉取镜像

```
拉取镜像指令：docker pull 镜像名:标签名
如果没有 :标签名 则默认下载为最新版本Copy
```

## 3.4、配置国内镜像加速器

配置阿里云镜像：需要注册一个阿里云账号

```
进入阿里云后台，搜索镜像加速器，点击镜像中心——>镜像加速器
就可以看到自己的 加速器地址
根据操作文档进行操作即可！

// CentOS为例
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://x6w3x27y.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart dockerCopy
```

# 第四章、Docker容器操作

## 4.1、查看容器

```
查询正在运行的容器：docker ps
                    CONTAINER ID：容器 ID
                    IMAGE ：镜像
                    COMMAND：初始命令
                    CREATED：创建日期
                    STATUS：容器状态
                    PORTS：端口号
                    NAMES：容器名字
查看所有的容器（启动与未启动的容器）：docker ps -a
查看最后一次运行的容器：docker ps –l
查看停止的容器：docker ps -f status=exitedCopy
```

## 4.2、创建容器

```
创建容器：docker run [OPTIONS] 镜像名:标签名
		OPTIONS：-i 表示交互式运行容器（就是创建容器后，马上会启动容器，并进入容器 ），通常与 -t 同时使用
						 -t 启动后会进入其容器命令行, 通常与 -i 同时使用; 加入 -it 两个参数后，容器创建就能登录进去。						     即分配一个伪终端。
             --name 为创建的容器指定一个名称。
             -d 创建一个守护式容器在后台运行，并返回容器ID。这样创建容器后不会自动登录容器，
                如果加 -i 参数，创建后就会运行容器。
             -v 表示目录映射, 格式为： -p 宿主机目录:容器目录
                注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。
             -p 表示端口映射，格式为： -p 宿主机端口:容器端口Copy
```

- 创建交互式容器（创建容器后，马上启动容器，并进入容器）
  /bin/bash 是linux中的命令解析器,会进入到容器里面命令行 （也可以省略不写,会自动生成）
  指令：docker run -it –name=mycentos centos:7 /bin/bash
- 退出容器：exit
- 退出但不停止当前容器：Ctrl + p + c

## 4.3、启动与停止容器

如果使用Ctrl + p + c退回当前容器后，不能使用exit退出容器。

```
1、启动已经运行过的容器：docker start 容器名称|容器id
2、启动所有已经运行过的容器：docker start `docker ps -a -q`
3、停止正在运行的容器：docker stop 容器名称|容器id
4、强制停止正在运行的容器：docker kill 容器名称|容器id
5、docker stop `docker ps -a -q`Copy
```

## 4.4、创建守护式容器

创建一个容器，不需要立即进入容器命令行窗口，而是后台运行的容器。

```
创建守护式容器：docker run -id --name=mycentos2 centos:7Copy
```

## 4.5、登录容器

```
登录已经启动的容器方：docker exec -it 容器名称|容器id /bin/bashCopy
```

## 4.6、退出守护式容器

```
exit 针对通过 docker exec 进入的容器，只退出但不停止容器Copy
```

## 4.7、拷贝宿主机与容器的文件

```
将宿主机文件拷贝到容器内：docker cp 要拷贝的宿主机文件或目录 容器名称:容器文件或目录
从容器内文件拷贝到宿主机：docker cp 容器名称:要拷贝的容器文件或目录 宿主机文件或目录

【所有操作都是在宿主机上执行，容器内不可执行！】
  
例如：docker cp /daemon.json 298c83a3db6a:/
     docker cp  298c83a3db6a:/daemon.json /Copy
```

## 4.8、数据目录挂载

**我们可以在创建容器的时候，将宿主机的目录与容器内的目录进行映射，这样我们就可以通过修改宿主机某**
**个目录的文件从而去影响容器。使用 -v 选项**

```
docker run -id -v /宿主机绝对路径目录:/容器内目录 --name=容器名 镜像名Copy
```

## 4.9、数据目录挂载只读(Read-Only)权限

```
docker run -id -v /宿主机绝对路径目录:/容器内目录:ro --name=容器名 镜像名Copy
```

## 4.10、查看容器内部细节

```
docker inspect 容器名|容器IDCopy
```

docker inspect 命令查询的结果为JSON格式，也可以查询指定内容，比如**查询容器IP地址**

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名Copy
```

## 4.11、删除容器

**只能删除停止的容器**

```
只能删除停止的容器 ： docker rm 容器名称 | 容器ID
删除所有容器：docker rm `docker ps -a -q`Copy
```

# 第五 章实战部署应用

## 5.1、MySQL部署

```
1、拉取mysql镜像：
docker pull mysql:5.7
  
2、创建mysql镜像：
docker run -id --name=docker_mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
   -p 代表端口映射，格式为 宿主机映射端口:容器运行端口
   -e 代表添加环境变量， MYSQL_ROOT_PASSWORD 是 root 用户的登陆密码
     
3、进入MySQL容器，登录MySQL
docker exec -it docker_mysql /bin/bash

4、远程连接mysql:使用宿主机的IP地址+宿主机映射端口(第二步)

5、连接超时原因：
      如连接不上，则查看宿主机防火墙有没关闭或者是上面暴露端口号配置是否正确
      查看状态： systemctl status firewalld
      关闭： systemctl stop firewalld
      开机禁用： systemctl disable firewalld
      
      查看阿里云服务器是否配置防火墙Copy
```

## 5.2、Redis部署

```
1、拉取redis镜像
docker pull redis

2、创建redis容器
docker run -id --name=docker_redis -p 6379:6379 redis
Copy
```

## 5.3、Tomcat部署

```
1、拉取镜像
docker pull tomcat

2、创建tomcat容器
docker run -id --name=docker_tomcat -p 8181:8080 -v /usr/local/project:/usr/local/tomcat/webapps --privileged=true tomcat
  
-p 表示地址映射, 宿主机端口号:容器运行端口号
-v 表示地址映射, 宿主机目录:容器映射目录
--privileged=true 如果映射的是多级目录，防止有可能会出现没有权限的问题，所以加上此参数

3、测试目录挂载
  在宿主机/usr/local/project下创建test/mumu.jpg图片文件
  登录docker：docker exec -it docker_tomcat /bin/bash
  查看webapps下是否有新增文件
  浏览器打开: ip地址:8181/test/mumu.jpgCopy
```

## 5.4、创建RabbitMQ容器

```
1、拉取镜像(有管理页面)
docker pull rabbitmq:management

2、创建镜像:
  方式一：创建镜像（默认用户名密码）,远程连接端口5672，管理系统访问端口15672，默认账号密码:guest
  docker run -id --name=docker_rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
  
  方式二：启动镜像（设置用户名密码）
  docker run -id --name=docker_rabbitmq -e RABBITMQ_DEFAULT_USER=username -e
  RABBITMQ_DEFAULT_PASS=password -p 5672:5672 -p 15672:15672 rabbitmq:managementCopy
```

## 部署常见问题

```
1、docker端口映射或启动容器时报错Error response from daemon: driver failed programming external connectivity on end
解决方案：docker服务启动时定义的自定义链DOCKER由于某种原因被清掉，重启docker服务及可重新生成自定义链DOCKER
systemctl restart dockerCopy
```

# 第六章 备份与迁移

## 6.1、 容器保存为镜像

将我们创建的容器保存为镜像后，数据迁移时就可以直接下载镜像，不需要再重新进行下载、配置容器。

```
通过以下命令将容器保存为镜像:
    docker commit [-m="提交的描述信息"] [-a="创建者"] 容器名称|容器ID 生成的镜像名[:标签名]Copy
```

## 6.1.1、无目录挂载-容器保存为镜像

```
查询是否有挂载目录 docker inspect --format='{{.Mounts}}'容器名 
docker commit [-m="提交的描述信息"] [-a="创建者"] 容器名称|容器ID 生成的镜像名[:标签名]Copy
```

## 6.1.2、有目录挂载情况

**问题**：

```
如果Docker对容器挂载了数据目录, 在将容器保存为镜像时，数据不会被保存到镜像中Copy
```

**原因**：

```
因为宿主机与容器做了路径映射，再commit一个新的镜像时，该路径下的所有数据都会被抛弃，不会被保存到新镜像中。可通过 docker inspect --format='{{.Mounts}}' 镜像名查看是否有目录挂载Copy
```

**解决**：

```
目录挂载方法：
		先把在宿主机的数据备份在某个目录下，在 docker run 的时候使用-v参数将宿主机上的
目录映射到容器里的目标路径中（tomcat是 /usr/local/tomcat/webapps ，mysql是 var/lib/mysql ）
  
拷贝方法：
     先把在宿主机的数据备份在某个目录下，通过拷贝的方法 docker cp 将备份的数据复制进容
器里的目标路径中（tomcat是 /usr/local/tomcat/webapps ，mysql是 var/lib/mysql ）。Copy
```

- 第一步，将需要备份的数据保存到宿主机上
- 第二步，将容器保存为镜像
- 第三步，下载镜像
- 第四步，恢复数据，备份数据量如果大，就采用拷贝方法(先启动容器后拷贝,比如mysql)，如果备份量小的话，就可以采用目录挂载(启动容器的时候就数据同步,比如tomcat)

```
tomcat案例：
	查询数据保存的位置：docker inspect --format='{{.Mounts}}' docker_tomcat
  将宿主机数据进行备份：cp -rf /usr/local/project/ /usr/local/baseproject
  保存容器为镜像：docker commit docker_tomcat tomcat_new:1
  采用目录挂载创建容器：docker run -id --name=tomcat_new_test -p 8081:8080 -v \
/usr/local/baseproject:/usr/local/tomcat/webapps --privileged=true tomcat_new:1Copy
mysql案例：
	mysql容器创建时，会自动实现目录挂载，查询看docker inspect --format='{{.Mounts}}' docker_mysql
  将宿主机的目录进行备份： cp -rf /var/lib/docker/****/ /usr/local/baseproject
  保存容器为镜像：docker commit docker_mysql mysql_new:1
  利用保存的容器创建容器：docker run -id --name=docker_mysql_new -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql_new:1
  利用拷贝方法进行数据还原，容器中的数据目录为：/var/lib/mysql
  docker cp /mysql/ docker_mysql_new:/var/lib/
  重启mysql容器：docker restart docker_mysql_newCopy
```

## 6.2、 备份镜像

问题：如何将自己的镜像发给别人使用？

```
docker save -o /opt/mycentos.tar mycentos_new:1.1
-o 指定输出到的文件Copy
```

## 6.3、镜像恢复与迁移

将别人打包好的镜像，解压

```
docker load -i mycentos.tar
-i输入的镜像文件Copy
```

# 第七章、Dockerfile语法与实战

## 7.1、什么是Dockerfile

```
Dockerfile 用于构建一个新的Docker镜像的脚本文件，是由一系列命令和参数构成的脚本
构建新的镜像步骤：
	1、编写Dockerfile文件
	2、通过docker build命令生成新的镜像
	3、通过docker run命令运行Copy
```

## 7.2、Dockerfile语法规则

- 每条指令的保留字(关键字)都必须为大写字母且后面至少要有一个参数
- 执行顺序按从上往下执行
- ‘#’号用于注释
- 每条指令都会创建一个新的镜像层，并对镜像进行提交

## 7.3、Dockerfile 执行流程

- Docker 从基础镜像运行一个容器
- 执行每一条指定并对容器作出修改
- 执行类似 docker commit 的操作提交一个新的镜像层
- docker 再基于刚提交的镜像运行一个新容器
- 执行 Dockerfile 中的下一条指令直到所有指令都执行完成

## 7.4、Dockerfile 常用指令

查询docker官网的镜像，已centos7为例

```
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20200504" \
    org.opencontainers.image.title="CentOS Base Image" \
    org.opencontainers.image.vendor="CentOS" \
    org.opencontainers.image.licenses="GPL-2.0-only" \
    org.opencontainers.image.created="2020-05-04 00:00:00+01:00"

CMD ["/bin/bash"]Copy
```

| 指令(大写字母为保留字)                 | **作用**                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| **FROM** image_name:tag                | 基础镜像，基于哪个基础镜像启动构建流程                       |
| **MAINTAINER** user_name               | 镜像的构建者的姓名和邮箱地址等                               |
| **COPY** source_dir/file dest_dir/file | 和ADD相似，但是如果有压缩文件并不能解压                      |
| **ADD** source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是压缩文件，将自动解压       |
| **ENV** key value                      | 设置环境变量，可以写多条                                     |
| **RUN** command                        | 是Dockerfile的核心部分，可以写多条，运行到当行，就指定该命令 |
| **WORKDIR** path_dir                   | 设置工作目录，当创建容器后，命令终端默认登录目录，未指定则为/ |
| **EXPOSE** port                        | 当前对外暴露的端口号，使容器内的应用可以通过端口和外界交互   |
| **CMD** argument                       | Dockerfile中可以有多个CMD，但是只有最后一个会生效。在构建容器时，会被 docker run 后面指定的参数覆盖 |
| **ENTRYPOINT** argument                | 和CMD相似，但是并不会被docker run指定的参数覆盖，而是追加参数 |
| **VOLUME**                             | 将宿主机文件夹挂载到容器中                                   |

## 7.5、Dockerfile构建镜像实战

**案例构建JDK**

- 创建目录，将jdk包上传到指定目录

- 创建文件Dockerfile文件，并编辑文件

  ```
  #来自基础镜像
  FROM centos:7
    
  #指定镜像创建者信息
  MAINTAINER mengxuegu
    
  #切换工作目录 /usr/local
  WORKDIR /usr/local
    
  #创建一个存放jdk的路径
  RUN mkdir /usr/local/java
    
  #将jdk压缩包复制并解压到容器中/usr/local/java
  ADD jdk-8u171-linux-x64.tar.gz /usr/local/java
    
  #配置java环境变量
  ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
  ENV JRE_HOME $JAVA_HOME/jre
  ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
  ENV PATH $JAVA_HOME/bin:$PATH
    
  CMD ["/bin/bash"]Copy
  ```

- docker build [-f 指定Dockerfile所在路径与文件名] -t 生成的镜像名:标签名 .

  注意后边的 空格 和点 . 不要省略, . 表示当前目录
  -f 指定Dockerfile文件所在路径与文件名。如果未指定 -f 值，则找当前目录下名为 Dockerfile 的构建文件

# 第八章、本地镜像发布到阿里云仓库

登录阿里云控制台，搜索**容器镜像服务**配置镜像仓库，可以根据操作指南进行操作