---
layout: post
title: Spring Cloud 学习笔记（七）一Spring Cloud--Feign-advance
date: 2019/9/4 18:39:19   
categories: document
tag: Spring Cloud

---

* content
{:toc}


# 1.Feign的工作原理

通过上一章的入门案例，我们可以快速了解feign的基本使用，这章先介绍一下feign的工作原理：（划重点啦，最重要的部分。莫走神！）

1.在开发我服务应用时，在主程序的入口添加@EnableFeignClients注解开启Feign Client扫描，加载处理。根据Feign Client开发规范，定义接口并加入注解@FeignClient(value = "eureka-provider")

2.当程序启动时，会扫描所有的包路径，找到@FeignClient注解的类，并将这些信息注入Spring IOC容器当中。当定义的Feign接口方法被调用的时候，通过JDK的代理方式，来生成具体的RequestTemplate。当生成代理是，Feign会为每个接口方法创建一个RequestTemplate对象，该对象闯将了HTTP请求需要的全部信息，如请求参数名、请求方发等信息。

3.然后由RequestTemplate生成Request，然后把Request交给Client方法去处理，这里的Client可以使JDK原生的URLConnection、Apache的HTTP Client，也可以是OKHttp。最后Clien北封装到LoadBalanceClient类中，这个类结合Ribbon负载均衡发起服务间的调用。

# 2.Feign的基本功能

## 2.1FeignClient注解剖析
打开FeignClient的注解定义类：org.springframework.cloud.openfeign.FeignClient，我们可以看到FeignClient注解对应的属性如下图：

![](/styles/images/spring-cloud/feign/8.png)

FeignClient注解的常用属性解释如下：

- name:指定FeignClient的名称，如果项目使用的Ribbon，name属性会作为服务的名称，用于服务发现。

- url：url一般用于调试，可以手动指定@FeignClient调用的地址。

- decode404：当发生4040错误时，如果该字段为true，会调用decoder进行解码，否则抛出FeignException。

- configuration：Feign配置类，可以自定义Feign的Encode，Decoder，Loglevel,Contract.

- fallback:定义容错的处理类，当调用远程接口失败或者超时时，会调用对应的接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口。

- fallbackFactory：工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口通过容错逻辑，减少代码重复。

- path：定义当前FeignClient的统一前缀。

## 2.2Feign开启日志

**在上一章的基础之上，进行日志开启演示！！！！！**

Feign为每个FeignClient都提供了一个leign.Logger实例，可以在配置中开启日志。（共两种方式，）
**方式1.**在application.yml文件中配置日志输出，只需要在spring-cloud-hello-comsumer项目下的.yml文件中配置即可。

配置代码如下：

```
logging:
    level:
        com.sn.springcloudhellocomsumer.feign.FeignConsumerService: debug

```

然后在FeignConsumerService.java类中，添加内部配置类UserFeignConfig.java，并在@FeignClient注释中添加配置注解信息，如下代码：
```
@FeignClient(value = "eureka-provider",configuration = UserFeignConfig.class)
public interface FeignConsumerService extends FeignService {

}

class UserFeignConfig {
    @Bean
    public Logger.Level logger() {
        return Logger.Level.FULL;
    }
}
```

切记：

    配置类UserFeignConfig上也可添加@Configuraiton 注解，声明这是一个配置类；但此时千万别将该放置在主应用程序上下文@ComponentScan 所扫描的包中，否则，该配置将会被所有Feign Client共享（相当于变成了通用配置，其实本质还是Spring父子上下文扫描包重叠导致的问题），无法实现细粒度配置！
    个人建议：不加@Configuration注解，省得进坑。这种方式是细粒度配置。也可以粗粒度配置全局的注解

启动项目，访问连接：http://localhost:8887/helloUser，此时，当该Feign Client的方法被调用时，将会打印类似如下的日志：每访问一次，日志就会打印一次。

![](/styles/images/spring-cloud/feign/9.png)

**方式2.**第二种配置方式，是只在application.yml配置文件中进行配置。

从Spring Cloud Edgware开始，Feign支持使用属性自定义Feign。对于一个指定名称的Feign Client（例如该Feign Client的名称为eureka-provider），Feign支持如下配置项，这里我们把没有必要的配置先注释掉：

```

logging:
    level:
        com.sn.springcloudhellocomsumer.feign.FeignConsumerService: debug

feign:
  client:
    config:
      eureka-provider:
        connectTimeout: 5000  # 相当于Request.Options
        readTimeout: 5000     # 相当于Request.Options
        # 配置Feign的日志级别，相当于代码配置方式中的Logger
        loggerLevel: full
        # Feign的错误解码器，相当于代码配置方式中的ErrorDecoder
        errorDecoder: com.example.SimpleErrorDecoder
        # 配置重试，相当于代码配置方式中的Retryer
        # retryer: com.example.SimpleRetryer
        # 配置拦截器，相当于代码配置方式中的RequestInterceptor
        # requestInterceptors:
        # - com.example.FooRequestInterceptor
        # - com.example.BarRequestInterceptor
        decode404: false
```
当然在.yml文件当中配置号以后，不用在FeignConsumerService文件中进行代码配置了。FeignConsumerService类代码回复原来的样子如下：

```
//@FeignClient(value = "eureka-provider",configuration = UserFeignConfig.class)
@FeignClient(value = "eureka-provider")
public interface FeignConsumerService extends FeignService {

}

/*
class UserFeignConfig {
    @Bean
    public Logger.Level logger() {
        return Logger.Level.FULL;
    }
}
*/
```

启动后，浏览器访问连接：http://localhost:8887/helloUser，此时，当该Feign Client的方法被调用时，将会打印类似如下的日志：

![](/styles/images/spring-cloud/feign/10.png)


如果使用了Java代码配置Feign，同时又使用了配置属性配置Feign，那么使用配置属性的优先级更高。配置属性配置的方式将会覆盖Java代码配置。


**Feign配置自定义【通用配置】**

上面讨论了如何配置特定名称的Feign Client，那么如果想为所有的Feign Client都进行日志配置。**首先代码方式**在@EnableFeignClients 注解上有个defaultConfiguration 属性，我们可以将默认配置写成一个类，然后用defaultConfiguration 来引用，例如在启动类上添加注解属性defaultConfiguration=UserFeignConfig.calss：
```
@EnableFeignClients(defaultConfiguration = UserFeignConfig.class)
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaConsumer {

	public static void main(String[] args) {
		SpringApplication.run(EurekaConsumer.class, args);
	}

	//添加负载均衡
	@Bean
	@LoadBalanced
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}

}
```

或者直接在.yml文件当中配置全局日志即可：
```
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```

## 2.3Feign开启压缩

开启请求和响应GZIP压缩，可以提高通信效率，一些场景下，我们可能需要对请求或响应进行压缩，此时可使用以下属性启用Feign的压缩功能
```
feign:
    compression:
          request:
              enabled: true #开启请求的GZIP压缩
              #配置压缩支持的MIME TYPE 用于支持的媒体类型列表，默认是text/xml、application/xml以及application/json
              mime-types: text/xml,application/xml,application/json
              min-request-size: 2048 #用于设置请求的最小阈值，默认是2048
          response:
              enabled: true #开启响应的GZIP压缩
```

# 3.Feign的实战运用

在web开发中Spring mvc 是支持GET请求方法直接绑定POJO进行传值的，但是Feign的实现并未覆盖所有的Spring mvc的功能，比如Feign不支持GET请求的多参数POJO类型参数传递，目前解决的方式如下：
0.使用POST请求替换GET请求传POJO类型的参数；
1.把POJO拆成一个个单独的属性放在参数里；
2.把方法参数标称map传递
3.使用get传递@RequestBody，但是吃方式违反Restful规范。
4.自定义Reign拦截器的方式处理（网上大佬给的意见）。

## 3.1 Feign的GET请求
我们先看一下GET请求传参普通的调用方式，代码如下：
```
@Api(description = "用户接口")
@RestController
@RequestMapping("/demoController")
public class applicationConsumerController {

    @Autowired
    FeignConsumerService feignConsumerService;

    @Autowired
    FeignService feignService;

@RequestMapping(value="/helloString",method = RequestMethod.GET)
    public String findUserById(@RequestParam("id") String id){
        return feignService.helloString(id);
    }

```
```
public interface FeignService {

 @RequestMapping(value = "/helloString", method = RequestMethod.GET)
    String helloString(@RequestParam("id")String id);
}

//实现类
@RestController
public class FeignServiceImpl implements FeignService{

    @Override
    public String helloString(String id) {
        User user = new User();
        if ("123".equals(id)) {

            user.setAge(20);
            user.setId("123");
            user.setUserName("你好");
        }
        return user.getUserName();
    }
}

```
如下图，在消费者系统引入swagger ui插件辅助测试， 找到测试url/demoController/helloString，输入id：123 点击try it out 如下图1，请求成功，返回结果如图2.可见已经通过feign获取到成产者系统的服务。

![](/styles/images/spring-cloud/feign/advance/1.png)
图1
![](/styles/images/spring-cloud/feign/advance/2.png)
图2

如果直接使用GIT 请求传入POJO入参，我们一起看一下会出现什么样的结果。
先看代码：
```
    @ApiOperation(value = "GET请求参数POJO测试" ,  notes="GET请求参数POJO测试")
    @RequestMapping(value ="/addUser",  method = RequestMethod.GET,consumes = MediaType.APPLICATION_JSON_VALUE)
    public String addUser2(User user){
        return feignConsumerService.helloUser(user);

```
如图3，输入user，POJO入参，点击请求，发生400异常如图4，后台没有报错，页面会提示错误信息为：
Required request body is missing: public java.lang.String com.sn.springcloudhellocomsumer.applicationConsumerController.addUser2(com.sn.springcloudhelloprovider.Fegin.User)

![](/styles/images/spring-cloud/feign/advance/3.png)
图3，
![](/styles/images/spring-cloud/feign/advance/4.png)
图4

可以看到参数并没有传入到请求的方法里，这就是前面提到的:
Feign的不支持GET请求传POJO参数，而SpringMVC是支持的，我们可以看出feign并没有完全实现springMvc的所有功能。
遇到图4这种情况我们只需要把GET请求改成POST请求即可，（当然也有别的方法解决，对get请求写一个拦截器进行转换。），下面我们看一下post请求

## 3.2 Feign的POST请求

fiegn的post请求支持传入pojo类型的参数。
代码如下：

```
    @ApiOperation(value = "post请求参数POJO测试" ,  notes="post请求参数POJO测试")
    @RequestMapping(value ="/addUserpost",  method = RequestMethod.POST,consumes = MediaType.APPLICATION_JSON_VALUE)
    public String addUser3(User user){
        return feignConsumerService.helloUserpost(user);
    }

//服务提供端接口
@RequestMapping(value = "/helloUserPost", method = RequestMethod.POST)
    String helloUserpost(@RequestBody User user);

//服务提供端接口实现方法
    @Override
    public String helloUserpost(User user) {
        return "success";
    }

```

启动服务，访问swagger的url /addUserpost ，如图5输入POJO参数，点解请求，如图6 请求成功。
![](/styles/images/spring-cloud/feign/advance/5.png)
图5
![](/styles/images/spring-cloud/feign/advance/6.png)
图6

**注意：**
1.消费者系统请求方式要和服务生产者系统对应的服务请求方式要保持一致，要么都是GET请求，要么都是PSOT请求。
2.当然无论是get请求还是post请求，都支持单个属性传参，这里就不在一一试验了，有心的 同学可以自己尝试一下。

## 3.3 Figen支持GET请求传POJO类型的参数

本节主要讲第3章开始提到方法4.自定义Reign拦截器的方式处理
通过实现Feign的RequestInterceptor方法apply，来实现拦截feign的GET请求方法多参数传递，拦截器代码如下：

```

@Component
public class FeignRequestInterceptor implements RequestInterceptor {
    private static final Logger log = LoggerFactory.getLogger(FeignRequestInterceptor.class);

    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public void apply(RequestTemplate template) {
        System.out.println("   --->>> 这是自定义拦截器   <<<---  ");
// feign 不支持 GET 方法传 POJO, json body转query
        if (template.method().equals("GET") && template.body() != null) {
            try {
                JsonNode jsonNode = objectMapper.readTree(template.body());
                template.body((String) null);

                Map<String, Collection<String>> queries = new HashMap<>();
                buildQuery(jsonNode, "", queries);
                template.queries(queries);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private void buildQuery(JsonNode jsonNode, String path, Map<String, Collection<String>> queries) {
        // 叶子节点
        if (!jsonNode.isContainerNode()) {
            if (jsonNode.isNull()) {
                return;
            }
            Collection<String> values = queries.get(path);
            if (null == values) {
                values = new ArrayList<>();
                queries.put(path, values);
            }
            values.add(jsonNode.asText());
            return;
        }
        // 数组节点
        if (jsonNode.isArray()) {
            Iterator<JsonNode> it = jsonNode.elements();
            while (it.hasNext()) {
                buildQuery(it.next(), path, queries);
            }
        } else {
            Iterator<Map.Entry<String, JsonNode>> it = jsonNode.fields();
            while (it.hasNext()) {
                Map.Entry<String, JsonNode> entry = it.next();
                if (StringUtils.hasText(path)) {
                    buildQuery(entry.getValue(), path + "." + entry.getKey(), queries);
                } else {  // 根节点
                    buildQuery(entry.getValue(), entry.getKey(), queries);
                }
            }
        }
    }
}
```

当然这种方式 自己定义的拦截器，调试可能会出现很多莫名其妙的问题。（我这边出现了，调试很久没调试成功，有时间和精力的同学可以自己尝试一下。）

## 3.4 Feign的文件上传


最近工作比较忙，没时间更新博客了，趁着周末加班，工作不多，写一下。

看了网上一些实现上传文件，有消费者系统的pom文件要加两个jar依赖，如下图：

![](/styles/images/spring-cloud/feign/advance/7.png)

图7

亲自测了一下不加好像也可以，（我用的springCloud版本号是2.1.4.RELEASE），如果版本低的话不知道要不要加，可以自己测试一下

开始代码，消费者系统controller如下，实现与页面的交互。
```
    @ApiOperation(value = "测试上传文件" ,  notes="测试上传文件")
    @RequestMapping(value = "/fileUpload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public String fileUpload( MultipartFile file){

        System.out.println("file:"+file.toString());
        return feignConsumerService.fileUpload(file);
    }
```
FeignConsumerService类中，新加方法fileUpload，添加注解RequestMapping和MultipartSupportConfig内部类，实现文件上传服务。如下代码；

```

@FeignClient(value = "eureka-provider",configuration = FeignConsumerService.MultipartSupportConfig.class)
public interface FeignConsumerService  extends FeignService {

    @RequestMapping(value = "/fileUpload", method = RequestMethod.POST,produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public String fileUpload(@RequestPart(value = "file")MultipartFile multipartFile);

    @Configuration
    class MultipartSupportConfig {
        @Bean
        public Encoder feignFormEncoder() {
            return new SpringFormEncoder();
        }
    }

}
```

服务提供端，FeignService接口中添加方法 handleFileUpload，提供文件上传服务，并且对接口进行实现。
```
@PostMapping(value = "/uploadFile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public String handleFileUpload(@RequestPart(value = "file") MultipartFile file);


//handleFileUpload 接口实现方法
 @Override
    public String handleFileUpload(MultipartFile file) {
        return "调用成功返回的的文件名为："+file.getOriginalFilename();
    }

```
启动eureka注册中心，启动服务端，启动消费端，

访问：http://localhost:8887/swagger-ui.html#/

我们用POST方法实现文件上传，如图8，选择文件，点击请求如图9，返回服务端接口实现类，返回的文件名。

![](/styles/images/spring-cloud/feign/advance/8.png)

图8

![](/styles/images/spring-cloud/feign/advance/9.png)

图9

feign除了以上这些功能,还要其他的功能，如：feign调用传递token，feign文件下载等功能。以后有时间了慢慢更新，这次先到这里。