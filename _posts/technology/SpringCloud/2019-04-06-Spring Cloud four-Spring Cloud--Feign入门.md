---
layout: post
title: Spring Cloud 学习笔记（六）一Spring Cloud--Feign
date: 2019/4/6 16:07:35    
categories: document
tag: Spring Cloud

---

* content
{:toc}


在使用Spring Cloud开发微服务应用时，各种服务提供者都是已HTTP接口的形式对外提供服务，因此在服务消费者调用服务提供者的时候，底层通过HTTP Client的方式访问。其实最方便最优雅的方式是使用Spring Cloud Open Feign 进行服务间的调用。Spring Cloud对Feign进行了增强，使其支持Spring MVC的注解，并整合了Ribbon等，从而让Feign的使用更加方便。
# 1.Feign概述

## what is Feign

Feign是一个声明式的WEB Service客户端。使用Feign只需要创建一个个接口加上对应的注解，比如：FeignClient注解。Feign有可插拔的注解，包括Feign注解和JAX-RS注解。Feign也支持编码和解码，Spring Cloud Open Feign 对Feign进行了增强，支持Spring MVC注解。
Feign是一个声明式、模板化的HTTP客户端。在使用Spring Cloud中使用Feign，可以使用HTTP请求访问远程服务，就像调用本地方法一样快捷方便。开发者完全感觉不到这是在远程调用方法，更感知不道有HTTP请求，Feign具体特性如下：
1.可插拔的注解支持，包括Feign注解和JAX-RS注解；
2.Feign支持可插拔的HTTP编码和解码；
3.支持Hystrix和Fallback。
4.支持Ribbon的负载均衡。
5.支持HTTP请求和响应的压缩。
6.还提供了HTTP请求的模板，通过编程简单的接口，和注解就可以定义好HTTP请求的参数、格式、地址信息等。
Feig完全代理HTTP请求，使用过程中我们只需要依赖注解Bean，然后调用对应的方法传递参数即可。
Open Feign地址：https://github.com/OpenFeign/feign
Spring Cloud Open Feign地址：https://github.com/spring-cloud/spring-cloud-openfeign
# 2.Feign入门

## 2.1开始前的准备工作
在Feign实际案例开始之前，我们先笔记（五）搭建的项目进行简单的重构。（之前的项目创建的太随意了，决定对项目重构，具体架构还是不变，在原有的基础之上加了一个父工程统一管理这三个子工程。）
创建工程，springcloudhello
工程结构如下图：
![](/styles/images/spring-cloud/feign/1.png)

pom.xml文件为：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.sn</groupId>
	<artifactId>spring-cloud-hello</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>
	<name>spring-cloud-hello</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<modules>
		<module>spring-cloud-hello-comsumer</module>
		<module>spring-cloud-hello-eureka</module>
		<module>spring-cloud-hello-provider</module>
	</modules>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<dependency>
				<groupId>com.sn</groupId>
				<artifactId>spring-cloud-hello-eureka</artifactId>
				<version>0.0.1-SNAPSHOT</version>
			</dependency>
			<dependency>
				<groupId>com.sn</groupId>
				<artifactId>spring-cloud-hello-comsumer</artifactId>
				<version>0.0.1-SNAPSHOT</version>
			</dependency>
			<dependency>
				<groupId>com.sn</groupId>
				<artifactId>spring-cloud-hello-provider</artifactId>
				<version>0.0.1-SNAPSHOT</version>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```
   
创建3个子工程：
- spring-cloud-hello-comsumer
- spring-cloud-hello-eureka
- spring-cloud-hello-provider

三个子工程 pom.xml文件依次为：
spring-cloud-hello-comsumer  pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.sn</groupId>
		<artifactId>spring-cloud-hello</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>
	<groupId>com.sn</groupId>
	<artifactId>spring-cloud-hello-comsumer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-cloud-hello-comsumer</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>

		<dependency>
			<groupId>com.sn</groupId>
			<artifactId>spring-cloud-hello-provider</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>

		<!--添加feign依赖-->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
		</dependency>


	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```
application.yml配置文件为：
```
server:
  port: 8887

spring:
  application:
    name: eureka-consumer

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8810/eureka/,http://localhost:8811/eureka/

eureka-provider:
    ribbon:
        NFLoadBalancerRuleClassName: com.sn.springcloudhellocomsumer.MyRule #选择的负载均衡策略
        ConnectTimeout: 1000 #请求连接的超时时间
        ReadTimeout: 3000 #请求处理的超时时间
        OkToRetryOnAllOperations: true #对所有操作请求都进行重试
        MaxAutoRetriesNextServer: 2 #切换实例的重试次数
        MaxAutoRetries: 1 #对当前实例的重试次数

ribbon:
    eager-load:
        enabled: true #开启Ribbon的饥饿加载模式
        clients: eureka-provider, eureka-consumer #指定需要饥饿加载的客户端名称、服务名
```

 spring-cloud-hello-eureka pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.sn</groupId>
		<artifactId>spring-cloud-hello</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>
	<groupId>com.sn</groupId>
	<artifactId>spring-cloud-hello-eureka</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-cloud-hello-eureka</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>

	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```
application.yml 配置文件为：
```
---
server:
  port: 8810
spring:
  profiles: peerA
  application:
    name: eureka-server-A
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2:8811/eureka/
---
server:
  port: 8811
spring:
  profiles: peerB
  application:
    name: eureka-server-B
eureka:
  instance:
    hostname: peer2
  client:
    instanceInfoReplicationIntervalSeconds: 10
    serviceUrl:
      defaultZone: http://peer1:8810/eureka/
```

spring-cloud-hello-provider pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.sn</groupId>
		<artifactId>spring-cloud-hello</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>
	<artifactId>spring-cloud-hello-provider</artifactId>
	<name>spring-cloud-hello-provider</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>

	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```
application.yml 配置文件为：

```
spring:
  application:
    name: eureka-provider

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8810/eureka/,http://localhost:8811/eureka/
```
这样就把三个子项目，放进一个父项目统一管理，三个子项目公用的依赖统一在springcloudhello 的pom.xml文件中添加，子项目单独添加依赖的，在对应子项目的pom.xml中添加，比如spring-cloud-hello-comsumer 要用到feign依赖，只需要在comsumer的pom.xml中添加即可。
每个子工程的目录结构和前几节的一样，后面会具体给出。

## feign入门配置

1.首选子项目spring-cloud-hello-eureka，即Eureka 注册中心不变和之前一模一样，如下图：
![](/styles/images/spring-cloud/feign/2.png)

2.子项目spring-cloud-hello-provider，服务提供方，我们新建目录feign，并且新建接口类FeignService，以及其实现类FeignServiceImpl，目录结构如下图：

![](/styles/images/spring-cloud/feign/3.png)

FeignService和FeignServiceImpl 代码如下：
```
public interface FeignService {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    User  hello(@RequestParam("userName") String userName);
}
```
```
@RestController
public class FeignServiceImpl implements FeignService {
    @Override
    public User hello(String userName) {
        User user = new User();
        user.setAge(20);
        user.setId("123");
        user.setUserName("你好");

        System.out.println();
        return user;
    }
}
```
3.子项目 spring-cloud-hello-comsumer的目录结构如下图：
![](/styles/images/spring-cloud/feign/4.png)
新建目录feign 及接口类FeignConsumerService 并继承FeignService，代码如下：
```
@FeignClient(value = "eureka-provider")
public interface FeignConsumerService extends FeignService {

}
```
这里不需要实现任何方法，@FeignClient注解表示，需要找到提供服务的服务器名，

到此完成了feign的基础配置，由于前几节我们介绍了ribbon负载均衡，所以这里我们就采用自定义的负载均衡，
在applicationConsumerController类中写实现方法，先把FeignConsumerService 通过 @Autowired加载进来，然后写调用方法，启动系统，通过浏览器调用来实现feign+ribbon的负载均衡调用，代码如下：
```
@RestController
public class applicationConsumerController {

  @Autowired
  FeignConsumerService feignConsumerService;

    @RequestMapping("/helloUser")
    public User findUser(){

        return feignConsumerService.hello("");
    }
}
```
首选启动spring-cloud-hello-eureka，中的EurekaServiceA和EurekaServiceB，
然后启动spring-cloud-hello-provider工程中的EurekaProviderA和EurekaProviderB，
最后启动spring-cloud-hello-comsumer 中的EurekaConsumer
然后浏览器访问http://localhost:8811/ 如下图，所有服务均已注册到服务中心：
![](/styles/images/spring-cloud/feign/6.png)

然后访问：http://localhost:8887/helloUser，发现能正常调用提供方服务器，结果如下图
![](/styles/images/spring-cloud/feign/7.png)

我们多次访问链接http://localhost:8887/helloUser，会发我们自定义的ribbon负载均衡策略也已经生效，8081和8080均被命中。
![](/styles/images/spring-cloud/feign/5.png)

项目源码地址：[https://github.com/ZFCC/springcloudhello.git](https://github.com/ZFCC/springcloudhello.git)



，




