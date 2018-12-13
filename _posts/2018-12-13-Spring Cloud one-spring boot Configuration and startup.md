---
layout: post
title:  Spring Cloud 学习笔记一intiallije idea中配置spring-boot并启动
date: 2018/12/13 19:24:40 
categories: document
tag: Spring Cloud

---

* content
{:toc}




# Spring Cloud 那些事 

## Spring Cloud 学习笔记一intiallije idea中配置spring-boot并启动

### 一、基本配置

#### 1.1项目生成

首先给大家介绍一个项目构建的网站：https://start.spring.io/
（个人亲身体会非常好用）
附截图，选择你要使用的服务，最后Generate Project，生成项目包.zip文件，解压之后放在你的intellij idea工作目录下，打开idea，进行简单的配置。下面介绍怎么样配置。(此种方法可以省去，pom.xml文件的配置，直接生成)

![](https://i.imgur.com/xFQ3jt7.png)

#### 1.2基本配置

首先配置jdk，方法见截图

![](https://i.imgur.com/2yLyYkg.png)

配置maven，见图

 ![](https://i.imgur.com/OecVk0y.png)

然后给项目加载maven管理：见截图

 ![](https://i.imgur.com/iKoGdEL.png)

趁着maven加载jar包的时候，写一下博客。。。。。。。。
全部配置好以后，不报错的情况下，完成上面启动类的创建。
启动方式：

![](https://i.imgur.com/ey1RsDs.png)

控制台会有spring的大字样，说实话很漂亮！！！

 ![](https://i.imgur.com/zcBqhfP.png)

启动如果控制台报错Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could，别慌张，是因为你在1.1节构架项目的时候添加了数据库管理的服务，因为项目还没有配置数据的信息，所以会出现这种错误，如下图，

![](https://i.imgur.com/bAUOW6T.png)

解决方法就是，在@SpringBootApplication注解的后面加上如下代码：

`exclude = DataSourceAutoConfiguration.class`

注意导包哦！！，然后用maven工具clean一下，再次启动即可。

 ![](https://i.imgur.com/f3P60v1.png)

启动成功后，控制台打印日志信息如下图：可以看到spring boot启动后的魅力标志

 ![](https://i.imgur.com/GguAvyk.png)

启动成功后，可以去浏览器输入http://localhost:8080/， 因为我们尚未搭建前端组件，所以暂时是sping boot提供的报错页面，

 ![](https://i.imgur.com/RRsUhK0.png)

我们的第一个Spring Boot项目也完成了！


