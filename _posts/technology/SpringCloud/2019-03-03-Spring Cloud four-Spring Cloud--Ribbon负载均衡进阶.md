---
layout: post
title: Spring Cloud 学习笔记（五）一Spring Cloud--Ribbon负载均衡进阶
date: 2019/2/2 16:39:26  
categories: document
tag: Spring Cloud

---

* content
{:toc}


# 1.Spring Cloud--Ribbon与Nginx对比

## Nginx：服务器端负载均衡
Nginx是客户端所有请求统一交给Nginx，由Nginx进行实现负载均衡请求转发，属于服务器端负载均衡。既请求由Nginx服务器端进行转发。
Nginx 适合于服务器端实现负载均衡 比如 Tomcat ，
## Ribbon：客户端负载均衡
Ribbon是从eureka注册中心服务器端上获取服务注册信息列表，缓存到本地，然后在本地实现轮训负载均衡策略。既在客户端实现负载均衡。
Ribbon 适合与在微服务中 RPC 远程调用实现本地服务负载均衡，比如 Dubbo、SpringCloud 中都是采用本地负载均衡。


# 2.Ribbon策略

说负载均衡，肯定会说到负载均衡的策略，
LoadBalancerClient在初始化的时候，会通过ILoadBalance（BaseLoadBalancer是实现类）向Eureka注册中心获取服务注册列表，并且每10s一次向EurekaClient发送“ping”，来判断服务的可用性，如果服务的可用性发生了改变或者服务数量和之前的不一致，则从注册中心更新或者重新拉取。LoadBalancerClient有了这些服务注册列表，就可以根据具体的IRule来进行负载均衡。

常见的策略有：RandomRule表示随机策略、RoundRobinRule表示轮询策略、WeightedResponseTimeRule表示加权策略、BestAvailableRule表示请求数最少策略
如下图负载均衡的继承结构：

![](https://i.imgur.com/acdzHBo.png)

## 2.1 Ribbon负载均衡策略
Ribbon内置了一些负载均衡，下面就让我们看一个表格，来详细了解Ribbon内置的负载均衡都有哪些，并且每个负载均衡都应该适用于哪种场景。

|内置负载均衡规则类 |规则描述|
|:-------|:-------|
|RoundRobinRule|简单轮询服务列表来选择服务器。它是Ribbon默认的负载均衡规则。
|RandomRule|随机选择一个可用的服务器。
|BestAvailableRule|忽略那些短路的服务器，并选择并发数较低的服务器。
|ZoneAvoidanceRule|以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。
|WeightedResponseTimeRule|为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。
|Retry|重试机制的选择逻辑。
|AvailabilityFilteringRule|1.在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。可以通过修改配置loadbalancer.<clientName>.connectionFailureCountThreshold来修改连接失败多少次之后被设置为短路状态。默认是3次2.并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。并发连接数的上线，可以由客户端的<clientName>.<clientConfigNameSpace>.ActiveConnectionsLimit属性进行配置。

## 2.2 RandomRule随机策略源码
随机策略就是从服务器中随机选择一个服务器，RandomRule的实现主要代码如下：

![](https://i.imgur.com/FYjkbom.png)

# 3 负载均衡策略实战

## 3.1 默认的RoundRobinRule轮询负载均衡策略：

在这几个项目的pom.xml文件中添加Ribbon的依赖，然后在消费者的启动类中添加负载均衡代码，通过使用声明Bean并且通过注解@LoadBalanced开启负载均衡。Ribbon负载均衡默认的策略时轮询策略，开启之后无需增加任何配置即可使用轮询策略。代码如下：

![](https://i.imgur.com/sQVfSoD.png)

![](https://i.imgur.com/96uJdH7.png)

启动系统，我们多次刷新请求，可以看到8080和8081这两个服务器实例，被轮询调用。

![](https://i.imgur.com/wColQtA.png)

## 3.2 负载均衡策略的配置
实际应用中多采用application.yml和application.properties配置的方式，这里就不在介绍代码或者注解的方式进行负载均衡的配置。eureka-provider:为提供消费的服务器端的系统实例名称，ribbon：负载均衡，NFLoadBalancerRuleClassName为需要配置的负载均衡策略的名称，然后就是负载均衡所在的路径。如下代码配置的是随机策略：

```
eureka-provider:
    ribbon:
        NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```
配置好以后重新启动下客户端系统，我们多次刷新请求，可以看到8080和8081这两个服务器实例被随机命中，如下图：
![](https://i.imgur.com/0PAToFt.png)

## 3.3 Ribbon的饥饿加载(eager-load)模式 
**问题：**Spring Cloud的Ribbon或Feign来实现服务调用的时候，如果我们的机器或网络环境等原因不是很好的话，有时候会发现这样一个问题：我们服务消费方调用服务提供方接口的时候，第一次请求经常会超时，而之后的调用就没有问题了。

**原因：**造成第一次服务调用出现失败的原因主要是Ribbon进行客户端负载均衡的Client并不是在服务启动的时候就初始化好的，而是在调用的时候才会去创建相应的Client，所以第一次调用的耗时不仅仅包含发送HTTP请求的时间，还包含了创建RibbonClient的时间，这样一来如果创建时间速度较慢，同时设置的超时时间又比较短的话，很容易就会出现上面所描述的显现。

**解决：**解决方式就是，在yml文件当中添加配置，把饥饿加载模式打开，如下代码：enabled为开启的开关true表示开启，默认是关闭的false；clients为指定需要饥饿加载的客户端名称、服务名，这里要多说一句，标点符号后面一定要加空格，而且标点符号一定要用英文的！！！

```
ribbon:
    eager-load:
        enabled: true#开启Ribbon的饥饿加载模式
        clients: eureka-provider, eureka-consumer#指定需要饥饿加载的客户端名称、服务名
```
不配置饥饿加载，我们多次刷新请求，可以看出第一次调用服务器请求耗时是：7268s
如下图：这种个情况下如果我们的机器或网络环境等原因不是很好的话，第一次请求可能会导致超时。

![](https://i.imgur.com/iuN5X6g.png)

配置好饥饿加载，我们在重启一下客户端系统，我们多次刷新请求，可以看到第一期调用服务器请求耗时是：81s，有些时候会是200-300s，但是也远低于不配置饥饿加载时第一次请求耗时，所以配置饥饿加载可以避免第一次请求时因网络或者其他原因导致加载超时。
![](https://i.imgur.com/2lCAPa1.png)

## 3.4 Ribbon的超时重试配置
如果一个实例发生了故障而该情况还没有被服务治理机制及时的发现和摘除，这时候客户端访问该节点的时候自然会失败。

所以，为了构建更为健壮的应用系统，希望当请求失败的时候能够有一定策略的重试机制，而不是直接返回失败。这个时候就需要开发人员人工的来为上面的RestTemplate调用实现重试机制。配置如下：

```
eureka-provider:
    ribbon:
        ConnectTimeout: 1000 #请求连接的超时时间
        ReadTimeout: 3000 #请求处理的超时时间
        OkToRetryOnAllOperations: true #对所有操作请求都进行重试
        MaxAutoRetriesNextServer: 2 #切换实例的重试次数
        MaxAutoRetries: 1 #对当前实例的重试次数
```
## 3.5 自定义负载均衡
最后我们在扩展一下，如果ribbon没有自己想要的负载均衡策略，我们需要自己开发一份合适的负载均衡策略，那么该怎么做。

首先创建一个自己的负载均衡类MyRule.java，并且实现（implements） IRule接口；然后依赖ILoadBalancer服务，获取服务列表信息。重写choose方法，如下示例：自定义策略 命中8080的概率是20%  命中8081的概率是80%，具体代码如下：
```
public class MyRule implements IRule {

    private ILoadBalancer lb;

    //自定义策略 命中8080的概率是20%  命中8081的概率是80%
    @Override
    public Server choose(Object o) {

        Random random = new Random();
        Integer num = random.nextInt(10);//在0-9这10个随机数里取值
        System.out.print("这是自定义的规则:num="+num);
        //获取传输负载均衡器里所有的服务
        List<Server> servers = lb.getAllServers();
        if(num>7){//返回8080端口服务
            return chooseServerByPort(servers,8080);
        }
        //返回8081端口服务
        return chooseServerByPort(servers,8081);

    }
    private Server chooseServerByPort(List<Server> servers,Integer port){
        for (Server server : servers) {
            if(server.getPort() == port){
                return server;
            }
        }
        return null;
    }

    @Override
    public void setLoadBalancer(ILoadBalancer lb) {
        this.lb = lb;
    }

    @Override
    public ILoadBalancer getLoadBalancer() {
        return lb;
    }
}

```
代码开发完以后我们在.yml文件当中把负载均衡策略的配置改成我们自己的策略，如下图：

![](https://i.imgur.com/liphIRM.png)

这是重启一下客户端系统，我们多次刷新请求，可以看到命中8080的概率接近20% 命中8081的概率是80%
为了验证，我们请求了30次，8080被命中的概率是6次基本符合我们设定的策略。如下图:
![](https://i.imgur.com/t38l1WT.png)


最后结束前，把所有的配置都加进来，就是一个比较健全的ribbon负载均衡的配置方式，如下图：

![](https://i.imgur.com/N8MXnL3.png)

----------



