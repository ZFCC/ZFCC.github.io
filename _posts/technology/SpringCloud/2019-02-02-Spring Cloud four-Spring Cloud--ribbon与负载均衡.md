---
layout: post
title: Spring Cloud 学习笔记（四）一Spring Cloud--Ribbon与负载均衡
date: 2019/2/2 16:39:26  
categories: document
tag: Spring Cloud

---

* content
{:toc}


# 1.Spring Cloud--Ribbon与负载均衡

负载均衡（Load Ribbon）即利用特定方式将流量推到多个操作单元上的一种手段，它对系统的吞吐量与系统处理能力有着质的提升，毫不夸张的说，目前很多的企业都还没有用到负载均衡器或者负载均衡策略。提到负载均衡很多人第一个想到的Nginx，或许是LVS，其本质都是一样的：对流量的疏导。
Ribbon是Netflix发布的负载均衡器，它可以帮我们控制HTTP和TCP客户端的行为。只需为Ribbon配置服务提供者地址列表，Ribbon就可基于负载均衡算法计算出要请求的目标服务地址。

Ribbon默认为我们提供了很多的负载均衡算法，例如轮询、随机、响应时间加权等——当然，为Ribbon自定义负载均衡算法也非常容易，只需实现IRule 接口即可。

在Spring Cloud中，当Ribbon与Eureka配合使用时，Ribbon可自动从Eureka Server获取服务提供者地址列表，并基于负载均衡算法，选择其中一个服务提供者实例。下图展示了Ribbon与Eureka配合使用时的大致架构。

![](/styles/images/spring-cloud/4/1.png)

## 1.1Ribbon特性

Ribbon是一个在云服务中久经沙场的客户端IPC库，它提供以下的一些特性：
-     负载均衡
-     故障容错
-     在异步和动态的模型中支持多协议通讯(HTTP、TCP、UDP)
-     缓存与批处理

引入Ribbon依赖，可以去Ribbon的maven仓库获取，下面是一个maven引入示例：
```
    <dependency>
        <groupId>com.netflix.ribbon</groupId>
        <artifactId>ribbon</artifactId>
        <version>2.2.2</version>
    </dependency>
```

## 1.2Ribbon所包含的模块

- ribbon：在其他Ribbon模块和Hystrix上集成负载均衡、容错、缓存/批处理的api
- ribbon-loadbalancer：可以独立或与其他模块一起使用的负载均衡器的api
- ribbon-eureka：使用Eureka客户端为云提供动态服务器列表的api
- ribbon-transport：使用带有负载均衡功能的RxNetty支持HTTP、TCP和UDP协议的传输客户端
- ribbon-httpclient：构建在Apache HttpClient之上，与负载均衡器集成的REST客户端
- ribbon-example：提供了一些示例
- ribbon-core：客户端配置api和其他共享api


## 1.3负载均衡器的三大子模块

-     Rule：确定从列表返回哪个服务的逻辑组件
-     Ping：在后台运行的组件以确保服务的活跃度
-     ServerList：这可以是静态的或动态的。如果它是动态的（由DynamicServerListLoadBalancer使用），后台线程将在特定的时间间隔刷新和过滤列表

# 2.Ribbon负载均衡入门案例

前几章，我们搭建起了高并发的Eureka服务器和客户端，我们在高并发的Eureka服务器和客户端之间加上我们这节说要讲述的Ribbon负载均衡，项目需要集群部署，也可以利用Eureka和Ribbon做到这一点。我们这里创建两个Eureka服务器端（这里两个Eureka可以理解为一个注册中心，对于外界来说只是一个统一的注册服务中心），两个Eureka客户端(作为服务提供者)，一个Eureka客户端(作为服务调用者)，增加一个Ribbon负载均衡。如图：Ribbon可以自动从Eureka服务中心获取服务的提供者地址列表，并基于负载均衡算法选择其中一个服务提供者实例。
![](/styles/images/spring-cloud/4/2.png)

## 2.1添加依赖和文件配置

首先在这几个项目的pom.xml文件中添加Ribbon的依赖，代码如下

```
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
		</dependency>
```
然后选择Eureka所在项目的工程配置包的配置文件中application.yml添加ribbon-loadBalance服务配置信息，具体如下：

![](/styles/images/spring-cloud/4/3.png)

第一行为:ribbon-loadBalance服务 名称。
第二行为：该ribbon负载均衡服务 分别向两个eureka注册中心注册，同样由eureka统一管理，
这样启动eureka注册中心后再启动负载均衡，则可以在eureka注册中心管理界面发现ribbon-loadBalance服务。

## 2.2代码部分

1、在之前章节的项目案例基础之上添加ribbon负载均衡的demo。首先创建一个负载均衡启动类RibbonLoadBalanceApplication.java 具体的代码如下：

```
@SpringBootApplication
@EnableEurekaServer
public class RibbonLoadBalanceApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(RibbonLoadBalanceApplication.class).properties("server.port="+8888).run(args);
    }

    //添加负载均衡
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

2、在服务提供者的项目ApplicationProviderController.java中添加测试用的demo

```
@RestController
public class ApplicationProviderController {

//负载均衡测试demo
    @GetMapping("/add")
    public String addTest(Integer a, Integer b, HttpServletRequest request){
        return "Form port:"+request.getServerPort()+",result:"+ a+b;
    }
}

```

3、在服务消费者系统的项目中applicationConsumerController.java中添加调用服务提供者的测试demo，代码如下：

@RestController
public class applicationConsumerController {
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/add")
    public String add(Integer a, Integer b){
        String result = restTemplate.getForObject("http://eureka-provider/add?a="+a+"&b="+b, String.class);
        System.out.println(result);
        return result;
    }
}

到此为止，我们的demo已经建好。可以启动这些高并发的Eureka服务器和客户端项目，来测试我们的demo是否成功。

## 2.3Demo测试部分
分别启动EurekaServiceA和EurekaServiceB，然后启动负载均衡服务器，EurekaServiceB，启动完成和后我们在浏览器界面输入http://localhost:8810/或者http://localhost:8811/进入Eureka管理界面，如下图，会发现除了eureka-server-A:8810和eureka-server-B:8811相互注册意外，还有ribbon-loadBalance:8888的成功注册。

![](/styles/images/spring-cloud/4/4.png)

然后我们启动EurekaProviderA和EurekaProviderB两个服务提供者，刷新Eureka管理界面http://localhost:8810/，这会发现这两个服务提供者eureka-provider:8081和eureka-provider:8080已经注册成功。如下图

![](/styles/images/spring-cloud/4/5.png)

然后我们启动服务消费者EurekaConsumer，如下图启动成功

![](/styles/images/spring-cloud/4/6.png)

最后在浏览器端输入链接：http://localhost:8887/add?a=1&b=2，并多次刷新请求，由于我们在applicationConsumerController的方法中已经实现了打印输出，我们会发现后台控制器中已经打印出一些信息，如下图：

![](/styles/images/spring-cloud/4/7.png)

我们可以看出我们每次用服务消费者EurekaConsumer调用消费者信息时，都是会调用得到不同的消费者返回的信息。这是因为Ribbon默认使用轮询的方式访问源服务，开篇已经提到，Ribbon的负载均衡策略有很多种，轮询策略只是其中的一种，后面章节我们会介绍其他的策略，并演示怎样自定义配置策略。


----------

今天是春节前腊月二十八，还在公司上班的苦逼的我，忙完手头的活，总结一下最近Spring Cloud的学习情况。撸完这个博客收拾东西准备下班回家过年，最后祝各位程序猿、攻城狮们新的一年事业有成，年终奖拿到手软！！！

