---
title: Go 01、语言环境安装
date: 2022-03-23 15:13:47
categories:
- Golang
- 基础
tags:
- Go
---



# Go 01、语言环境安装

# 一、Go 安装包下载

Go语言中文网下载 https://studygolang.com/dl

根据不用的环境，安装不同的版本，这里已Windwos为例。

![image-20221021145820856](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021145820856.png)

下载完成后，得到 go1.19.2.windows-amd64.msi ，然后下一步下一步无脑安装即可。

![image-20221021150037436](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021150037436.png)

安装完成后，命令行输入 `go version` 查看是否安装成功，显示版本号则表示成功！

![image-20221021150447898](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021150447898.png)

# 二、配置环境变量

1、在系统变量中添加GOROOT，变量值填写GO的安装位置

![image-20221021150714279](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021150714279.png)

2、在系统变量中添加GOPATH，这个目录是用来存储后续代码的，变量值填写自已定的代码存储位置

![image-20221021151022037](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021151022037.png)

最好将用户变量中的GOPATH(这个是安装时自动生成的)也修改为我们新建的代码存储位置，

![image-20221021152043280](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021152043280.png)

3、在GOPATH中的工作目录下，创建bin、pkg、src 三个文件夹，用来存储代码。

![image-20221021151221769](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021151221769.png)

4、设置PATH环境变量，正常情况下，使用安装包 安装的，会自动将GO安装目录添加至PATH

![image-20221021151828058](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021151828058.png)

5、到此，环境变量就配置好了，可以使用命令 `go env` 查看配置是否成功！

![image-20221021152311977](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021152311977.png)

# 三、GOLAND编译器下载安装

官网：https://www.jetbrains.com/

![image-20221021152541968](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021152541968.png)

下载完成后，下一步下一步无脑安装即可！

![image-20221021153109129](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021153109129.png)

# 四、编写第一个Go代码

1、打开GOLAND编译器，然后New Project

![image-20221021160123327](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021160123327.png)

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello World") //打印输出
}

```

![image-20221021160744338](https://raw.githubusercontent.com/lukaixin0527/images/master/java-imgimage-20221021160744338.png)