---
layout: post
title: Spring Cloud 学习笔记（三）一Spring Cloud--Eureka项目集群之服务管理框架理解
date: 2019/1/26 15:23:11 
categories: document
tag: Spring Cloud

---

* content
{:toc}


# 1.Spring Cloud--Eureka项目集群之服务管理框架理解

紧跟上一篇博客，这里采用的也是上一篇博客的代码作为分析：《Spring Cloud 那些事之Spring Cloud 学习笔记二：SpringCloud之Eureka-Service集群搭建》。我们在搭建起了简单的单机模式Eureka项目之后，如果Eureka服务器和客户端不能满足高并发访问，项目需要集群部署，也可以利用Eureka做到这一点。我们这里创建两个Eureka服务器端，两个Eureka客户端(作为服务提供者)，一个Eureka客户端(作为服务调用者)，如图：

 ![](/styles/images/spring-cloud/3/1.png)

通过运行多个实例并让它们彼此注册，Eureka可以变得更有弹性和可用性。实际上，这是默认行为，所以需要做的就是将一个有效的serviceUrl添加到对等点。我们可以将多个对等点添加到一个系统中，只要它们之间至少有一条边连接，它们就会同步这些注册。如果对等体在物理上是分开的(在数据中心或多个数据中心之间)，那么系统就可以在原则上生存下来，以避免“大脑分裂”。由于资源限制，这里让服务运行在同一个主机上，通过修改hosts文件配置，模仿两个不同的主机。

搜素Windows的“记事本”应用，以管理员身份运行，文件-->打开，找到C:\Windows\System32\drivers\etc目录，打开里面的“hosts”文件，在文件末尾为自己的主机添加两个虚拟的域名：
```
127.0.0.1 peer1 peer2
```

# 2.创建两个Eureka服务器端

找到我们上一篇博客编写的EurekaServiceA和EurekaServiceB项目，修改application.yml文件，配置如下：
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

由于需要启动的是两个项目，所以这里配置一下两个项目的名称。再配置启动环境，方便我们作为项目的启动条件，打开EurekaServiceA的class类和EurekaServiceB的class类，修改main方法。如下代码
```
package com.sn.springCloud101;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServiceA {

	public static void main(String[] args) {
		
		new SpringApplicationBuilder(EurekaServiceA.class).profiles("peerA").run(args);
	}
}
 ```
```
package com.sn.springCloud101;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServiceB {

	public static void main(String[] args) {
		new SpringApplicationBuilder(EurekaServiceB.class).profiles("peerB").run(args);
	}
}
 ```

之后分别运行EurekaServiceA和EurekaServiceB的main()方法，启动项目，项目EurekaServiceA启动过程中会报错，但是不影响我们启动项目，报错的原因是eureka-server-A会将自己的服务注册到eureka-server-B，但是eureka-server-B还未启动，我们可以在浏览器访问http://localhost:8810，能够进入Eureka的后台管理界面，说明项目已经启动成功。eureka-server-B项目是不会报错的，它能成功将自己注册到eureka-server-A。最后访问http://localhost:8811我们两个管理后台都能看见注册的服务信息：

 ![](/styles/images/spring-cloud/3/2.png)

 ![](/styles/images/spring-cloud/3/3.png)

# 3.创建两个Eureka客户端(提供者)

找到我们上一篇博客编写的EurekaProviderA再创建一个EurekaProviderB.java文件，代码如下：

 ```
package com.sn.springCloud102;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class EurekaProviderA {

	public static void main(String[] args) {
//		SpringApplication.run(EurekaProviderA.class, args);

		new SpringApplicationBuilder(EurekaProviderA.class).properties("server.port="+8080).run(args);
	}
}
  ```
 ```
package com.sn.springCloud102;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class EurekaProviderB {

	public static void main(String[] args) {
//		SpringApplication.run(EurekaProviderB.class, args);

		new SpringApplicationBuilder(EurekaProviderB.class).properties("server.port="+8081).run(args);
	}

 ```

然后修改application.yml文件，为服务提供者再提供一个服务器端的注册地址，让我们的服务提供者可以将自己的服务注册到两个服务器端，配置如下：

 ```
spring:
  application:
    name: eureka-provider

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8810/eureka/,http://localhost:8811/eureka/

 ```

为了能够让我们待会儿要创建的Eureka客户端(服务调用者)可以区分到底是从哪一个Eureka客户端(服务提供者)获取到返回值的，我们改造一下ApplicationProviderController类，设置UserName，用以展示对应请求的路径(包含端口号)，用端口号来鉴别调用的是哪一个服务：如下代码：

 ```
package com.sn.springCloud102;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;

@RestController
public class ApplicationProviderController {

    @RequestMapping(value = "/search/{id}", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public User searchUser(@PathVariable String id, HttpServletRequest sr){
        User user = new User();
        user.setId(id);
        user.setAge(20);
        user.setPassWord("123");
        user.setUserName("EurekaProvider--"+sr.getRequestURL().toString());

        return user;

    }
}
  ```

运行EurekaProviderA和EurekaProviderB类的main()方法，启动两个项目；项目成功启动之后，我们在浏览器分别访问http://localhost:8080/search/A和http://localhost:8081/search/B，能够得到对应的响应结果，如下：

 ![](/styles/images/spring-cloud/3/4.png)

 ![](/styles/images/spring-cloud/3/5.png)

然后访问我们的Eureka服务器端，http://localhost:8810或者http://localhost:8811，也能够看到服务被注册到了服务器端。

 ![](/styles/images/spring-cloud/3/6.png)

 ![](/styles/images/spring-cloud/3/7.png)

# 4.创建一个Eureka客户端(服务调用者)

找到我们上一篇博客编写的EurekaConsumer，修改application.yml文件，为服务调用者再提供一个服务器端的注册地址，让我们的服务调用者可以将自己的服务注册到两个服务器端，再将端口修改了，避免冲突导致的不能启动项目，配置如下：
```
server:
  port: 8888

spring:
  application:
    name: eureka-consumer

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8810/eureka/,http://localhost:8811/eureka/
 ```
为了让结果更加明显，我们对applicationConsumerController.java进行简单的修改，代码如下

```
package com.sn.springCloud103;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

import javax.servlet.http.HttpServletRequest;
import java.security.PrivateKey;
import java.util.List;

@RestController
public class applicationConsumerController {

    @Autowired
    private DiscoveryClient discoveryClient;

    //添加负载均衡配置
    @Bean
    @LoadBalanced
    public org.springframework.web.client.RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    @RequestMapping(value = "/consumer/{id}",method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public String testConsumer(@PathVariable String id, HttpServletRequest request) {

        List<ServiceInstance> list = discoveryClient.getInstances("STORES");
        System.out.println(discoveryClient.description());
        System.out.println("discoveryClient.getServices().size()+" + discoveryClient.getServices().size());
//打印EurekaProvider客户端的服务信息
        for (String str : discoveryClient.getServices()) {
            System.out.println("services " + str);
            List<ServiceInstance> serviceInstances = discoveryClient.getInstances(str);
            for (ServiceInstance si : serviceInstances) {
                System.out.println("    services:" + str + ":getHost()=" + si.getHost());
                System.out.println("    services:" + str + ":getPort()=" + si.getPort());
                System.out.println("    services:" + str + ":getServiceId()=" + si.getServiceId());
                System.out.println("    services:" + str + ":getUrl()=" + si.getUri());
                System.out.println("    services:" + str + ":getMetadata()=" + si.getMetadata());
            }
        }
        String string = "";


        System.out.println(id);
        String flag =id;
        if (id.equals("A")) {
            System.out.println(id);
            RestTemplate rt = new RestTemplate();
//            RestTemplate rt = getRestTemplate();
            string= rt.getForObject("http://localhost:8080/search/A", String.class);
            return string;
//        return rt.getForObject("http://EUREKA-PROVIDER/search/A", String.class);
        }
        if (id.equals("B")) {
            System.out.println(id);
            RestTemplate rt = new RestTemplate();
//            RestTemplate rt = getRestTemplate();
            string= rt.getForObject("http://localhost:8081/search/B", String.class);
            return string;
        }
        //            RestTemplate rt = getRestTemplate();
//            return rt1.getForObject("http://localhost:8081/search/A", String.class);
        return string;
    }
}


```
服务消费者需要的配置就完成了，然后我们运行EurekaConsumer类的main()方法，成功启动之后，访问http://localhost:8888/consumer/A和http://localhost:8888/consumer/B可以看到，我们的服务调用者是依次去调用了8080和8081的服务。

 ![](/styles/images/spring-cloud/3/8.png)

 ![](/styles/images/spring-cloud/3/9.png)

这时刷新eureka-service-A和eureka-service-B页面可以发现也有变化

 ![](/styles/images/spring-cloud/3/10.png)

 ![](/styles/images/spring-cloud/3/11.png)

至此搭建完成！！！
完整代码地址如下：（三个项目分别对应三个服务，1.0.1对应EurekaService；1.0.2对应EurekaProvider；1.0.3对应EurekaConsumer）
 [https://github.com/ZFCC/springCloud1.0.1 ](https://github.com/ZFCC/springCloud1.0.1 )
 [https://github.com/ZFCC/springCloud1.0.2](https://github.com/ZFCC/springCloud1.0.2)
[https://github.com/ZFCC/springCloud1.0.3](https://github.com/ZFCC/springCloud1.0.3)

精彩内容，请见下回分解


