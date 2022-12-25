---
title: Spring Cloud Alibaba
date: 2022-03-23 15:13:47
categories:

- 微服务
tags:
 - Spring Cloud Alibaba
---



# Spring Cloud Alibaba

# 一、Spring Cloud Alibaba

## 1、简介

Spring官网：https://spring.io/projects/spring-cloud-alibaba
GitHub：https://github.com/alibaba/spring-cloud-alibaba
GitHub中文文档：https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md
版本说明：https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E

阿里云脚手架构建项目：https://start.aliyun.com/bootstrap.html

![image-20221010171006045](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221010171006045.png)





# 二、Nacos

## 1、Nacos简介

官方：https://nacos.io/zh-cn/

开发文档：https://nacos.io/zh-cn/docs/what-is-nacos.html

Nacos /nɑ:kəʊs/ 是 Dynamic Naming and Configuration Service的首字母简称，一个更易于构建云原生应用的**动态服务发现**、**配置管理**和**服务管理**平台。

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

## 2、Nacos安装与启动

安装环境：Linux、JDK1.8+

1、下载安装包 https://github.com/alibaba/nacos/releases 目前下载为最新版本2.1.1，上传至 **/usr/local** 目录下

![image-20221010163621567](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221010163621567.png)

2、解压安装 

```shell
tar -zxvf nacos-server-2.1.1.tar.gz
```

3、启动服务器

启动命令(standalone代表着单机模式运行，非集群模式):

```shell
sh startup.sh -m standalone
```

如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：

```shell
bash startup.sh -m standalone
```

4、关闭服务器

```shell
sh shutdown.sh
```

5、单机模式下整合mysql

在0.7版本之前，在单机模式时nacos使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源能力，具体的操作步骤：

- 1.安装数据库，版本要求：5.6.5+
- 2.初始化mysql数据库，数据库初始化文件：mysql-schema.sql  (在安装目录 /nacos/conf/nacos-mysql.sql)
- 3.修改conf/application.properties文件，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码。

```
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://11.162.196.16:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=nacos_devtest
db.password=youdontknow
```

再以单机模式启动nacos，nacos所有写嵌入式数据库的数据都写到了mysql

6、登录Nacos界面  http://localhost:8848/nacos/  初始化密码：nacos/nacos

![image-20221010165719647](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221010165719647.png)

![image-20221010165736108](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221010165736108.png)

## 3、SpringCloudAlibaba整合Nacos

### 1、Nacos Config 配置中心

官网文档：https://github.com/alibaba/spring-cloud-alibaba/blob/2.2.x/spring-cloud-alibaba-examples/nacos-example/nacos-config-example/readme-zh.md

1. 首先，修改 pom.xml 文件，引入 Nacos Config Starter。

   ```
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   ```

2. 在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常使用。

   SpringBoot中配置文件的加载是存在优先级顺序的，bootstrap优先级高于application

   bootstrap.yml

   ```yml
   server:
     port: 8080
   spring:
     application:
       name: order-pay-01
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848 #Nacos服务注册中心地址（本机的写localhost）
         config:
           server-addr: 127.0.0.1:8848 #Nacos作为配置中心地址（本机的写localhost）
           file-extension: yml #指定yml格式配置
   ```

   application.yml

   ```yml
   spring:
       profiles:
           active: dev #表示开发环境
   ```

3. 使用 @EnableDiscoveryClient 注解开启服务注册与发现功能

   ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class ProviderApplication {
   
    	public static void main(String[] args) {
    		SpringApplication.run(ProviderApplication.class, args);
    	}
    }
   ```

4. Nacos创建配置文件

   dataID在 Nacos Config Starter 中，dataId 的拼接格式如下

   ```
   ${prefix} - ${spring.profiles.active} . ${file-extension}
   ```

   - `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。

   - `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)

     **注意，当 active profile 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}`.`${file-extension}`**

   - `file-extension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension`来配置。 目前只支持 `properties`和`yaml` 类型。

   - `group` 默认为 `DEFAULT_GROUP`，可以通过 `spring.cloud.nacos.config.group` 配置。

   Nacos创建 **order-pay-01-dev.yml** 配置文件，格式为yaml。

   ![image-20221012113311898](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221012113311898.png)

5.  使用注解@RefreshScope 开启自动更新，在Nacos中修改后，立即更新到代码中。

   ```java
   @RefreshScope   //支持Nacos的动态刷新功能
   @RestController
   public class ConfigClientController {
   
       @Value("${config.info}")
       private String configInfo;
   
       @GetMapping("/config/info")
       public String getConfigInfo(){
           return configInfo;
       }
   }
   ```

6. 配置分组group和namespace命名空间

   修改配置文件

   ```yml
   server:
     port: 8080
   spring:
     application:
       name: order-pay-01
     cloud:
       nacos:
         discovery:
           server-addr: 8.142.5.146:8848 #Nacos服务注册中心地址（本机的写localhost）
         config:
           server-addr: 8.142.5.146:8848 #Nacos作为配置中心地址（本机的写localhost）
           file-extension: yml #指定yml格式配置
           group: ORDER_GROUP # 分组名称
           namespace: cf538543-4da9-4d39-8540-80a6c1c26029 # 命名空间ID
   ```

   ![image-20221012115205623](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221012115205623.png)

### 2、Nacos Discovery 注册中心

官网文档：https://github.com/alibaba/spring-cloud-alibaba/blob/2.2.x/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example/readme-zh.md

1. 首先，修改 pom.xml 文件，引入 Nacos Discovery Starter。

   ```shell
   <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

2. 在应用的 /src/main/resources/application.properties 配置文件中配置 Nacos Server 地址

   ```yml
    spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   ```

3. 使用 @EnableDiscoveryClient 注解开启服务注册与发现功能

   ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class ProviderApplication {
   
    	public static void main(String[] args) {
    		SpringApplication.run(ProviderApplication.class, args);
    	}
    }
   ```

# 三、OpenFeign服务接口调用

官网文档：https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/#spring-cloud-openfeign

Feign是一个声明式的web服务客户端，让编写web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可。

### 1、Feign能干什么

Feign旨在使编写Java Http客户端变得更容易。
前面在使用Ribbon +RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通第都会针对每个微服务自行封装一些客户端类来包装
这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个
Feign注解即可），即可完成对服务提供方的接口鄉定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

### 2、Feign集成了Ribbon

利用Ribbon维护了Payment的服务列表信息，并日通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过feign只需要定义
服务绑定接口旦以声明式的方法，优雅而简单的实现了服务调用

### 3、Feign与OpenFeign的区别

![feign](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgfeign.png)

### 4、基本使用

服务列表

```
消费者:OrderPay01
生产者:OrderServer01
生产者:OrderServer02
流程：消费者——调用——>生产者
```

a.引入pom文件(消费者)

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

b.主启动类添加注解 @EnableFeignClients （消费者）

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
@SpringBootApplication
@EnableDiscoveryClient // 通过注解 开启nacos服务注册
@EnableFeignClients // 整合openFeign
public class OrderPay01Application {
    public static void main(String[] args) {
        SpringApplication.run(OrderPay01Application.class, args);
    }
}
```

c.消费者创建接口，接口内容复制生产者controller方法

```java
@Component
@FeignClient(value = "order-server") // 消费者注册到Nacos的服务名称
public interface OrderService {

    // 使用Feign接口调用远程服务的方法时，定义各参数绑定，@PathVariable、@RequestParam、@RequestHeader等可以指定参数属性，
    // 在Feign中绑定参数必须通过value属性来明确指明具体的参数名，不然会抛出异常。
    // 如果使用  public String echo(@PathVariable String string),没有(value = "string")则会出现异常
    
    @GetMapping(value = "/order/{string}")
    public String echo(@PathVariable(value = "string") String string);
}
```

生产者controller层

```java
@RestController
public class OrderController {
    @GetMapping(value = "/order/{string}")
    public String echo(@PathVariable String string) {
        return string + "___端口9090";
    }
}
```

![image-20221012101255924](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221012101255924.png)

### 5、OpenFeign超时控制

默认消费者调用生产者，超时1秒后会报超时异常,可以通过配置改变超时控制

在消费者配置文件中新增

```java
#没提示不管它，可以设置
#指的是建立连接后从服务器读取到可用资源所用的时间
ribbon.ReadTimeout:5000
#指的是建立连接使用的时间，适用于网络状况正常的情况下，两端连接所用的时间
ribbon.ConnectTimeout:5000
```

### 6、OpenFeign日志打印功能

```java
NONE:默认的，不显示任何日志
BASIC：仅记录请求方法、URL、响应状态码以及执行时间
HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息
HEADERS：除了HEADERS中定义的信息之外，还有请求和响应的正文以及元数据
```

1、消费者服务下 配置日志bean

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel(){
        //打印最详细的日志
        return Logger.Level.FULL;
    }
}
```

2、消费者服务下 添加配置文件

```java
#写你们自己的包名
logging.level.top.lukaixin.service.OrderService: debug
```

3、重启服务，访问接口

![image-20221012103902240](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221012103902240.png)

# 四、Sentinel

## 1、Sentinel简介

官网：https://sentinelguard.io/zh-cn/index.html

开发文档：https://sentinelguard.io/zh-cn/docs/introduction.html

整合Spring CloudAlibaba(推荐)   https://github.com/alibaba/spring-cloud-alibaba/wiki/Sentinel

Github详细文档(推荐)    https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D

**官方控制台说明**： https://github.com/alibaba/Sentinel/wiki/%E6%8E%A7%E5%88%B6%E5%8F%B0

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 是面向分布式、多语言异构化服务架构的流量治理组件，主要以流量为切入点，从**流量路由**、**流量控制**、**流量整形**、**熔断降级**、**系统自适应过载保护**、**热点流量防护**等多个维度来帮助开发者保障微服务的稳定性。

## 2、Sentinel安装与启动

1、下载Jar文件 https://github.com/alibaba/Sentinel/releases 目前最新版本1.8.5 上传至/usr/local下

![image-20221012160357158](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221012160357158.png)

2、启动控制台，执行 Java 命令 `java -jar sentinel-dashboard.jar`完成 Sentinel 控制台的启动。 控制台默认的监听端口为 8080。Sentinel 控制台使用 Spring Boot 编程模型开发，如果需要指定其他端口，请使用 Spring Boot 容器配置的标准方式

使用如下命令启动控制台：

```
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
```

其中 `-Dserver.port=8080` 用于指定 Sentinel 控制台端口为 `8080`。

3、访问 [http://localhost:8080](http://localhost:8080/) 页面 账户密码: sentinel / sentinel

![image-20221012161011872](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221012161011872.png)

## 3、SpringCloudAlibaba整合Sentinel

https://blog.csdn.net/weixin_55730337/article/details/124664284

1. 首先，修改 pom.xml 文件

   ```
   <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   ```

2. 编写配置文件

   ```java
   server:
     port: 18080
   spring:
     application:
       name: order-pay-01
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848 #Nacos服务注册中心地址（本机的写localhost）
         config:
           server-addr: localhost:8848 #Nacos作为配置中心地址（本机的写localhost）
           file-extension: yml #指定yml格式配置
           group: DEFAULT_GROUP # 分组名称
       sentinel:
         transport:
           port: 8719 # 默认8719端口，假如被占用了会自动从8719端口+1进行扫描，直到找到未被占用的 端口
           dashboard: localhost:8080 #配置Sentin dashboard地址（改成自己的服务器ip地址，本地用localhost‍）
   ```

3. 编写测试接口，并模拟请求

   ```java
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   @RestController
   public class SentinelDemoController {
       
       @GetMapping(value = "/test")
       public String test() {
           return String.valueOf(System.currentTimeMillis());
       }
   }
   ```

4. 登录sentinel控制台查看

![image-20221013103142806](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221013103142806.png)

**注意：请确保 Sentinel 控制台所在的机器时间与自己应用的机器时间保持一致，否则会导致拉不到实时的监控数据。**

## 4、Sentinel簇点链路

![image-20221013105602164](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221013105602164.png)

## 5、Sentinel流控规则

官方文档：https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6

当 QPS 超过某个阈值的时候，则采取措施进行流量控制。流量控制的效果包括以下几种：**快速失败**、**Warm Up**、**匀速排队**。

### 1、流控效果——快速失败

**快速失败**（`RuleConstant.CONTROL_BEHAVIOR_DEFAULT`）方式是默认的流量控制方式，当QPS超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出`FlowException`。这种方式适用于对系统处理能力确切已知的情况下，比如通过压测确定了系统的准确水位时。

返回 `Blocked by Sentinel (flow limiting)`

![image-20221013134953471](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221013134953471.png)

### 2、流控效果——Warm Up

Warm Up（`RuleConstant.CONTROL_BEHAVIOR_WARM_UP`）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮

​	默认coldFactor为**3**，即请求QPS从  **threshold(单机阈值) /3** 开始，经过 设置的**预热时长** 后逐渐升至设定的QPS阈值。

​	例如下图中，系统初始化的阈值为 10/3 =3，刚开始请求的时候阈值为3，经过5秒后阈值慢慢恢复到10

​	**注意：**如果请求持续，则5秒后QBS一直为10，如果中间间断，则又恢复预热，一开始3，慢慢恢复至10

![](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221013140116690.png)

![86e1-ecacff8aab51](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img86e1-ecacff8aab51.png)

### 3、流控效果——排队等待

匀速排队（`RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER`）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。![68292442-d4af3c00-00c6-11ea-8251-d0977366d9b4](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img68292442-d4af3c00-00c6-11ea-8251-d0977366d9b4.png)

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

> 注意：匀速排队模式暂时不支持 QPS > 1000 的场景。

例如下图中，每秒只接受1个请求，多余的请求排队，如果排队的请求超出1000毫秒，则失效。

![image-20221013143507608](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221013143507608.png)

### 4、流控模式——关联



### 5、流控模式——链路