---
title: Spring Cloud
tags: Spring Cloud
categories: 框架
---
## Spring Cloud简介

- Spring Cloud是基于Spring Boot的
- 为维服务体系开发中的架构问题，提供了一整套的解决方案——服务注册与发现，服务消费，服务保护与熔断，网关，分布式调用追踪，分布式配置管理等。
- Spring Cloud是关注全局的服务治理框架
- Spring Cloud是关注全局的服务治理框架
- Spring Cloud是关注全局的服务治理框架
- 完整的应用从数据存储开始垂直拆分成多个不同的服务，每个服务都能独立部署，独立维护，独立扩展
- RestFul方式进行服务调用

## 快速简化版本

### 服务注册与发现

#### 创建服务注册中心

Eureka作为服务注册与发现的组件

启动一个注册中心，只需要在springboot工程的启动application类上加：@EnableEurekaServer

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run( EurekaServerApplication.class, args );
    }
}
```

eureka上一个高可用的组件，它没有后端缓存，每一个实例注册之后需要想注册中心发送心跳(因此可以在内存中完成)，在默认情况下erureka server也是一个eureka client，必须要指定一个server。eureka server 的配置文件

```xml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

spring:
  application:
    name: eurka-server
```

通过eureka.client.registerWithEureka：false和fetchRegistry：false来表明自己是一个eureka server.

#### 创建一个服务提供者 (eureka client)

当client向server注册时，它会提供一些元数据，例如主机和端口，url，主页等。Eureka server 从每个client实例接收心跳消息。 如果心跳超时，则通常将该实例从注册server中删除。

依赖

```xml
   <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```



通过注解@EnableEurekaClient 表明自己是一个eurekaclient.

```java
@SpringBootApplication
@EnableEurekaClient
@RestController
public class ServiceHiApplication {

    public static void main(String[] args) {
        SpringApplication.run( ServiceHiApplication.class, args );
    }

    @Value("${server.port}")
    String port;

    @RequestMapping("/hi")
    public String home(@RequestParam(value = "name", defaultValue = "forezp") String name) {
        return "hi " + name + " ,i am from port:" + port;
    }

}

--------------------- 
作者：方志朋 
来源：CSDN 
原文：https://blog.csdn.net/forezp/article/details/81040925 
版权声明：本文为博主原创文章，转载请附上博文链接！
```

仅仅@EnableEurekaClient是不够的，还需要在配置文件中注明自己的服务注册中心的地址，application.yml配置文件如下：

```xml
server:
  port: 8762

spring:
  application:
    name: service-hi

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
--------------------- 
作者：方志朋 
来源：CSDN 
原文：https://blog.csdn.net/forezp/article/details/81040925 
版权声明：本文为博主原创文章，转载请附上博文链接！
```

### 服务消费者Feign(Finchley版本)

Spring cloud有两种服务调用方式，一种是ribbon+restTemplate，另一种是feign。

#### Feign简介

使用Feign，只需要创建一个接口并注解。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。

Feign的起步依赖spring-cloud-starter-feign、Eureka的起步依赖spring-cloud-starter-netflix-eureka-client、Web的起步依赖spring-boot-starter-web

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>
--------------------- 
作者：方志朋 
来源：CSDN 
原文：https://blog.csdn.net/forezp/article/details/81040965 
版权声明：本文为博主原创文章，转载请附上博文链接！
```

```xml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8765
spring:
  application:
    name: service-feign
```

定义一个feign接口，通过@ FeignClient（“服务名”），来指定调用哪个服务。比如在代码中调用了service-hi服务的“/hi”接口，代码如下：

```
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

在Web层的controller层，对外暴露一个"/hi"的API接口，通过上面定义的Feign客户端SchedualServiceHi 来消费服务。代码如下：

```
@RestController
public class HiController {


    //编译器报错，无视。 因为这个Bean是在程序启动的时候注入的，编译器感知不到，所以报错。
    @Autowired
    SchedualServiceHi schedualServiceHi;

    @GetMapping(value = "/hi")
    public String sayHi(@RequestParam String name) {
        return schedualServiceHi.sayHiFromClientOne( name );
    }
}
```

启动程序，多次访问http://localhost:8765/hi?name=forezp,浏览器交替显示：

hi forezp,i am from port:8762

hi forezp,i am from port:8763

### 断路器（Hystrix）(Finchley版本)

Netflix开源了Hystrix组件，实现了断路器模式，SpringCloud对这一组件进行了整合。 在微服务架构中，一个请求需要调用多个服务是非常常见的，如下图：

![](https://user-images.githubusercontent.com/24977343/57746971-207bbc80-7706-11e9-84a7-6474c8bf9d92.png)



较底层的服务如果出现故障，会导致连锁故障。当对特定的服务的调用的不可用达到一个阀值（Hystric 是5秒20次） 断路器将会被打开。

![](https://user-images.githubusercontent.com/24977343/57746996-36897d00-7706-11e9-83ed-7bdfd372cb35.png)

断路打开后，可用避免连锁故障，fallback方法可以直接返回一个固定值。

#### Feign中使用断路器

Feign是自带断路器的

feign.hstrix.enabled=true

基于service-feign工程进行改造，只需要在FeignClient的SchedualServiceHi接口的注解中加上fallback的指定类就行了：

```java
@FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

SchedualServiceHiHystric需要实现SchedualServiceHi 接口，并注入到Ioc容器中，代码如下：

```java
@Component
public class SchedualServiceHiHystric implements SchedualServiceHi {
    @Override
    public String sayHiFromClientOne(String name) {
        return "sorry "+name;
    }
}
```

启动四servcie-feign工程，浏览器打开http://localhost:8765/hi?name=forezp,注意此时service-hi工程没有启动，网页显示：

### 路由网关

在微服务架构中，需要几个基础的服务治理组件，包括服务注册与发现、服务消费、负载均衡、断路器、智能路由、配置管理等，由这几个基础组件相互协作，共同组建了一个简单的微服务系统。一个简答的微服务系统如下图：

![](https://user-images.githubusercontent.com/24977343/57752660-7ce9d680-771c-11e9-9b75-06e8bd9a8342.png)

A和B可以相互调用。

在Spring Cloud微服务系统中，一种常见的负载均衡方式是，客户端的请求首先经过负载均衡（zuul、Ngnix），再到达服务网关（zuul集群），然后再到具体的服。，服务统一注册到高可用的服务注册中心集群，服务的所有的配置文件由配置服务管理（下一篇文章讲述），配置服务的配置文件放在git仓库，方便开发人员随时改配置。

## Spring Consul

- 启动命令：	consul agent -dev