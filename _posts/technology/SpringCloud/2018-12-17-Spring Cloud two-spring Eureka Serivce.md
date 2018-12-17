---
layout: post
title: Spring Cloud 学习笔记（二）一SpringCloud之Eureka-Service集群搭建
date: 2018/12/13 19:24:40 
categories: document
tag: Spring Cloud

---

* content
{:toc}


# Spring-boot之Eureka-Service集群搭建
Eureka是Spring Cloud提供的众多模块中的一个，它主要用于服务管理，对服务进行分布式和集群，Eureka提供了服务器端和客户端，服务端致力于服务列表的维护和后台管理，客户端分为提供者（provider）和消费者（consumer）。

每一个客户端将自己注册到服务器端之后，其余的客户端就能检索发现对应的服务，所以提供者和消费者是针对调用关系来说的，实际上每一个客户端都可以同时作为提供者和消费者。如图简易的集群结构所示，一个Eureka服务器端，两个Eureka客户端(两个作为提供者，一个作为消费者)，来演示具体的调用过程。
 ![](https://i.imgur.com/jSJ906R.jpg)

## 1.创建Eureka服务器端

使用https://start.spring.io/  链接创建三个springCloud的maven工程，在Selected Dependencies选择需要的服务，至少包涵以下服务web，Eureka Server， Ribbon如图所示，把三个工程解压到同一个工作目录下，启动intellij idea把工作目录切换到该目录，然后给每个工程添加maven管理（具体步骤详见：intiallije idea中配置spring-boot并启动文档）。如图所示。
![](https://i.imgur.com/xjQFGs8.png)

![](https://i.imgur.com/eB40Azg.png)

![](https://i.imgur.com/PvYQscW.png)
 
打开resource目录新建文件application.yml，在文件中进行服务配置，配置如下：
 ```
server:
  port: 8810

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

 ```

其中port: 8810表示启动占用端口号，可以随意配置。
Hostname：localhost服务的名称，也可以随意配置
registerWithEureka 表示是否将自身注册到服务器，false表示不将自身注册，true将自身注册，这里不注册。
fetchRegistry ：如果为true，启动时会报警。
http://${eureka.instance.hostname}:${server.port}/eureka/：注册中心默认端口就是8761，可通过这句来定义选择其他端口号（网上这么说的，但是亲测，此句不要也可以选择其他端口号）。

完成了上面的配置和代码编写，就可以启动我们的eureka-server了，运行EurekaService的main()方法，访问http://localhost:8810，在浏览器看到我们的后台管理页面，Eureka服务器端就搭建完成了。

 
## 2.创建Eureka客户端的提供者（Eureka-Provider）
同第2节类似，打开springCloud1.02工程中的application.java文件，修改文件名为EurekaProvider，其他不变。

 ![](https://i.imgur.com/mCPKss4.png)

创建provider案例：在同一个目录下创建ApplicationProviderController.java和user.java如图所示

 ![](https://i.imgur.com/8lnJ5mx.png)

在resource目录下配置application.yml
```
spring:
  application:
    name: eureka-provider

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8810/eureka/
```
其中defaultZone: http://localhost:8810/eureka/：表示向Eureka-server注册服务。

如图： 
![](https://i.imgur.com/8dspM1w.png)

运行EurekaProvider的main()方法，启动成功后访问http://localhost:8080/search/1，这里没有配置端口，就默认使用了8080端口，在浏览器端可以看到控制器返回的数据。
 
![](https://i.imgur.com/yuNytej.png)

刷新访问http://localhost:8810，可以看到我们的服务提供者已经注册到Eureka服务器了。
![](https://i.imgur.com/a0S1r9g.png)
 
## 3.创建Eureka客户端的消费者
同第三节一样，打开springCloud1.03工程中的application.java文件，修改文件名为EurekaConsumer，其他不变。
 ![](https://i.imgur.com/0JJZxdf.png)

创建consumer消费者案例：在同一个目录下创建ApplicationConsumerController.java,住已配置负载均衡，RestTemplate是Spring框架里面提供的，它可以自动配置去使用Ribbon。要创建一个负载均衡的RestTemplate，使用创建RestTemplate的Bean组件(@Bean)并使用@LoadBalanced限定符。
 ![](https://i.imgur.com/OAjd1wC.png)

在src/main/resources下创建application.yml，写明自己的服务名称和需要注册到那个Eureka服务器上，并配置属于自己的端口。
```
server:
  port: 8811

spring:
  application:
    name: eureka-consumer

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8810/eureka/

 ```

![](https://i.imgur.com/j1UQ6Lj.png)

运行EurekaConsumer的main()方法，启动成功后访问http://localhost:8811/consumer，在浏览器端，我们可以看见eureka-consumer成功调用了eureka-provider的方法。
![](https://i.imgur.com/QyCsdVf.png) 

再次访问http://localhost:8810，我们可以看到eureka-server控制台上维护着两个Eureka客户端

 ![](https://i.imgur.com/hYsoIhU.png)

自此，集群配置成功。
