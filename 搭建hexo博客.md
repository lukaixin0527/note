---
title: 搭建Hexo博客
date: 2022-03-23 15:13:47
categories:
- 运维
tags:
- hexo
- 博客
---



# 搭建Hexo博客

## 一、Hexo是什么？

hexo 可以理解为是基于node.js制作的一个博客工具，简单来说利用hexo框架可以快速搭建一个属于自己的博客网站

 关于hexo的介绍，官网上的介绍很详细了。https://hexo.io/

## 二、本地快速搭建hexo博客

### 1、安装Node.js

进入node.js官网，下载LTS长期支持版

[![img](https://lukaixin0527.github.io/images/nodejs.jpg)](https://lukaixin0527.github.io/images/nodejs.jpg)

### 2、下载hexo

打开命令行

切换到root权限：sudu su

查看nodejs的版本和npm的版本：node -v npm -v

需要借助nmp安装包，但是国内很慢，所以需要借助国内的nmp包管理器 npm install -g cnpm –registry=[https://registry.npm.taobao.org](https://registry.npm.taobao.org/)

检测cnpm是否安装成功，输入cnpm -v

借助cnpm全局安装hexo cnpm install -g hexo-cli 然后验证 hexo -v

### 3、本地搭建Hexo博客

本地建立一个空文件夹 blog 所有博客文件全部在blog中生成，如果出错，删除重来

进入blog 文件

sudu hexo init 初始化一个hexo博客

hexo s 启动

### 4、编写博客文章

hexo n “我的第一篇博文” 会生成到source/_posts文件下

hexo clean 清除缓存

hexo g 重新生成 然后访问 [http://localhost:4000](http://localhost:4000/) 就可以了

我一般会用Typora软件写博客

```
安装 代码高亮插件 
// 1.卸载 Hexo 默认自带的 Markdown 渲染器
cnpm un hexo-renderer-marked --save
// 2.安装 hexo-renderer-markdown-it 插件
cnpm i hexo-renderer-markdown-it --save
// 3.修改_config配置文件
markdown: 'commonmark'
// 4.关闭hexo自带的渲染插件
highlight:
  enable: false // 关闭
  line_number: true
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: falseCopy
```

### 5、修改Hexo主题

直接在官网获取更多主题 https://hexo.io/themes/ ，选择自己喜欢的主题，获取git路径

比如我下载的主题是 yilia

git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia

修改配置文件_config.yml

theme: landscape 修改为 theme: yilia

然后重新执行命令就可以了

hexo clean

hexo g

hexo s

## 三、将博客部署到github

一般我们的博客会部署到github上，这样就可以远端访问了

创建github账号(以我的github为例 lukaixin0527)，并且创建一个库 库名格式必须为 ： lukaixin0527.github.io

下载一个部署插件,进入blog文件夹下 cnpm install –save hexo-deployer-git

修改设置，修改_config.yml 到文件的最底部 repo:为github路径

deploy:

type: git

repo: https://github.com/lukaixin0527/lukaixin0527.github.io.git

branch: master

最后部署到远端 hexo d 输入github账号和密码就可以了

如果提示403，说明没有切换github账号信息，找到本地的账号密码信息删除即可。

如果成功了，直接访问xxxxx.github.io就可以访问了，比如我的github博客网址 lukaixin0527.github.io

## 四、域名绑定

首先需要购买一个域名

我是在阿里云买的域名 lukaixin.com

然后需要实名认证后才可以使用

### 1、解析域名

 就是把网站域名解析到你所购买的网站空间(云服务器)IP上，其实这里说的解析就是一个绑定和指向关系，网站域名指向空间IP地址。

 简单来说，就是访问购买的域名就可以访问到github上的博客网址

 接下来开始解析域名

 这是我的域名，已经实名认证了，DNS服务器状态显示 正常

[![img](https://lukaixin0527.github.io/images/aliyunhoutai.jpg)](https://lukaixin0527.github.io/images/aliyunhoutai.jpg)

 点击解析设置——添加记录，选择CNAME类型(将一个域名指向另一个域名的类型)

 到这里，域名解析的完成了第一步

### 2、github设置

 在github仓库首页，选择自己的仓库 (lukaixin0527.github.io)

 找到 create new file ，点击新建文件

 文件名称命名为 **CNAME**

 内容填写自己的域名 比如我的 [www.lukaixin.com](http://www.lukaixin.com/)

 最后点击commit new file 提交文件就可以

### 3、配置成功

 到这里域名就配置成功了，直接访问域名，就可以访问了

## 五、配置SSL安全证书

**HTTP**是明文传输协议，传输内容容易被嗅探和篡改。

而**HTTPS**，即**HTTP over SSL/TLS**,是添加了一层**SSL(Secure Sockets Layer，安全套接层)\**，或者是**TLS(Transport Layer Security,***\*传输层安全协议)\**，所以**HTTPS**就可以视为**HTTP**和**SSL/TLS**协议的组合。

**HTTPS**能做到良好的保密性(防嗅探)，真实性(防篡改)，完整性(防域名劫持和域名欺骗)。

### 1、购买证书

进入阿里云首页，直接搜索SSL证书，点击购买，进入证书选择页面

因为我的证书是在阿里云购买的，所有可以享受一年免费的证书(嘿嘿~)

[![img](https://lukaixin0527.github.io/images/zhengshuxuanze.jpg)](https://lukaixin0527.github.io/images/zhengshuxuanze.jpg)

### 2、证书申请

购买证书成功后，进入证书详情，点击【证书申请】，填写相关信息，注意填写正确的域名

因为我的证书和域名都在阿里云购买，所有可以享受自动DNS验证，不需要自己去验证了

点击下一步，再点击验证，到这里就ok了，只需要耐心等待审核就可以了。

[![img](https://lukaixin0527.github.io/images/zhengshushenqing.jpg)](https://lukaixin0527.github.io/images/zhengshushenqing.jpg)

### 3、配置成功

等待证书配置成功后，就可以安全访问了。

比如我的域名：https://www.lukaixin.com/