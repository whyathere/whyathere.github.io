---
title: Spring Boot
tags: Spring Boot
categories: 框架
---
## Spring Boot简介

Spring Boot:名字字面意思就是spring的引导，就是用于启动spring，使得spring的学习和使用变得快速无痛。

内嵌web容器，由应用启动tomcat，而不是tomcat启动应用，来解决部署运行问题。

因为 Spring 的配置非常复杂，各种XML、 JavaConfig、hin处理起来比较繁琐。于是为了简化开发者的使用，从而创造性地推出了Spring boot，约定优于配置，简化了spring的配置流程。

### Spring Boot的功能

​	Spring Boot实现了自动配置，降低了项目搭建的复杂度。

​	Spring框架需要进行大量的配置，Spring Boot引入自动配置的概念，让项目设置变得很容易。它并不是用来替代Spring的解决方案，而是和Spring框架紧密结合用于提升Spring开发者体验的工具。同时它集成了大量常用的第三方库配置(例如Jackson, JDBC, Mongo, Redis, Mail等等)，Spring Boot应用中这些第三方库几乎可以零配置的开箱即用(out-of-the-box)，大部分的Spring Boot应用都只需要非常少量的配置代码，开发者能够更加专注于业务逻辑。

​	Spring Boot只是承载者，辅助你简化项目搭建过程的。如果承载的是WEB项目，使用Spring MVC作为MVC框架，那么工作流程和你上面描述的是完全一样的，因为这部分工作是Spring MVC做的而不是Spring Boot。

​	对使用者来说，换用Spring Boot以后，项目初始化方法变了，配置文件变了，另外就是不需要单独安装Tomcat这类容器服务器了，maven打出jar包直接跑起来就是个网站，但你最核心的业务逻辑实现与业务流程实现没有任何变化。

​	Spring Boot 是基于Spring4的条件注册的一套快速开发整合包。

### 依赖包简单说明

- spring-boot-starter-web：全站Web开发模块，包含嵌入式Tomcat、Spring MVC
- spring-boot-starter-test:通用测试模块，包含JUnit、Hamcrest、Mockito

Starter POMs：一系列轻便的依赖包，是一套一站式的Spring相关技术的解决方案。开发者在使用和整合模块时，不必再去寻找样例代码中的依赖配置来复制使用，只需要引入对对应的模块包即可。

### YAML文件

### Spring boot 热部署

在pom中加入devtools

## 记录日志

- Commons-logging or SLF4j

```java
private static final Log log = LogFactory.getLog(HelloWorld.class);

private static final Logger logger = LoggerFactory.getLogger(HelloWorld.class);
```

静态是因为在静态的方法里也会需要日志，一般是当前类的class。

- 日志级别：

TRACE<DEBUG<INFO<WARN<ERROR<FATAL

- Application.yml配置日志

## 使用CURL来测试API

在命令行

curl -v http://localhost:8080

如果是post delete 和put就不好通过浏览器来操作。

这里就可以使用命令行工具

GET：curl  http://localhost:8080

POST:  curl -H "Content-Type:application/json" -X POST —data '{"name":"World","age":"17"}' http://127.0.0.1:8080/

-H 指定http头里面的信息。requestBody就是—data里面的信息

DELETE：curl -X DELETE http://127.0.0.1:8080/

PUT: curl -H "Content-Type:application/json" -X PUT —data'{"name":"Person of Interest"}' http://127.0.0.1:8080/

或者POST Man

## RestController中获取请求的各种数

#### Get

```java
@GetMapping("/{id}")
public String getStr(@PathVariable int id){
    return id+"";
}
```

@PathVariable 这里指定的id表示会从路径中输入的{id}来取值。

#### POST

```java
@PostMapping
public UserDto insertOne(@RequestBody UserDto userDto){
    userDto.setAge(123);
    return userDto;
}
```

@RequestBody :将request的内容转换成UserDto

通过命令行来看

curl -H "Content-Type:application/json" -X POST --data '{"name":"西部实际","age":1}' http://localhost:8080/zero
返回   ：{"name":"西部实际","age":123}%

#### PUT

```JAVA
@PutMapping("/{name}")
public UserDto updateOne(@PathVariable String name,@RequestBody UserDto userDto){
    return userDto;
}
```

curl -H "Content-Type:application/json" -X PUT --data '{"name":"西部实际","age":1}' http://localhost:8080/zero

返回：{"name":"西部实际","age":1}%

DELETE

```java
@DeleteMapping("/{id}")
public Map<String,String> deleteOne (@PathVariable int id, HttpServletRequest request,
                                     @RequestParam(value = "delete_reason",required = false)String deleteReason) throws Exception{

    Map<String,String> result = new HashMap<>();

    result.put("message","101"+request.getRemoteAddr()+"删除原因:"+deleteReason);

    return result;

}
```

```
HttpServletRequest request
```

获取request，不需要加任何注解，spring boot会自动将request传进来，只要声明就可以了。

```
@RequestParam
```

这个注解只要request.getParamter可以取到就都可以取到。

### 上传和下载简单示例

```java
@PostMapping(value = "/{id}/photos",consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public void addPhoto(@PathVariable int id, @RequestParam("photo")MultipartFile imgFile) throws Exception{
    //保存文件
    FileOutputStream fos = new FileOutputStream("target/"+imgFile.getOriginalFilename());

    IOUtils.copy(imgFile.getInputStream(),fos);

    fos.close();

}
```

```java
consumes
```

指定传入的数据是什么格式的，request里面的数据。fromdata的形式

@RequestParam("photo")MultipartFile imgFile   多部分的文件。

```
IOUtils
```

可以将inputStream拷贝到outputStream

这样就将文件保存了。

 mac@MacBook-Pro-2  ~/Documents/zero/img  curl -v  -F "photo=@poi.png" http://127.0.0.1:8080/zero/12/photos
*   Trying 127.0.0.1...
*   TCP_NODELAY set
*   Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
> POST /zero/12/photos HTTP/1.1
> Host: 127.0.0.1:8080
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Length: 45416
> Expect: 100-continue
> Content-Type: multipart/form-data; boundary=------------------------7d06709df61f1d53
>
> < HTTP/1.1 100
> < HTTP/1.1 200
> < Content-Length: 0
> < Date: Tue, 19 Feb 2019 00:45:08 GMT
> <
* Connection #0 to host 127.0.0.1 left intact
   mac@MacBook-Pro-2  ~/Documents/zero/img 

```java
@GetMapping(value = "/{id}/icon",produces = MediaType.MULTIPART_FORM_DATA_VALUE)
public byte[] getIcon(@PathVariable int id) throws Exception{
    String iconFile = "src/test/resource/icon.jpg";
    InputStream is = new FileInputStream(iconFile);
    return IOUtils.toByteArray(is);
}
```

这里是使用的produces

### 对传入参数的校验

**原则**

- 不要相信前端传过来的数据
- 不能相信前端传递过来的任何数据
- 绝不要相信前端传过来的任何数据

```java
@PostMapping
public UserDto insertOne1(@RequestBody UserDto userDto){
    if (userDto == null){
        throw new RuntimeException("参数userDto不可为空");
    }
    if (userDto.getName() == null){
        throw new RuntimeException("电视剧名字不能为空");
    }
    return null;
}
```

比如这样写是没有问题的，但是代码行数比较多。比较繁琐。条件也比较单一

**简单办法**

Bean Validation

- JSR303
- Hibernate Validator

JSR303：一个java规范，对JavaBean进行验证的规范。是标准而不是具体的实现。

Hibernate Validator：实现bean的验证的实现。

在spring boot加入就可以使用

Bean Validation注解如下：

- @Null  验证对象是否为空		传了就错了
- @NotNull 验证对象是否为非空        不传就错了
- @Min 验证Number和String对象是否大等于指定的值
- @Max 验证Number和String对象是否小等于指定的值
- @Size  验证对象(Array,Collection,Map,String)长度是否在给定的范围之内   验证长度
- @Past 验证Date和Calendar对象是否在当前时间之前
- @Future 验证Date和Calendar对象是否在当前时间之后
- @AssertTrue 验证Boolean对象是否为true
- @AssertFalse 验证Boolean对象是否为false
- @Valid 级联验证注解   写在一个Bean前面的

还有验证邮箱和信用卡的

## Tomcat配置

通用的Servlet容器配置都以"server"作为前缀，而Tomcat特有配置都以"server.tomcat"作为前缀。

例子：

**配置Servlet容器:**

server.port= #配置程序端口，默认是8080

server.session-timeout= #用户会话session过期时间，以秒为单位

server.context-path= #配置访问路径，默认为/

**配置Tomcat**

server.tomcat.uri-encoding= #配置程序端口，默认为8080

server.tomcat.compression= #Tomcat是否开启压缩，默认为关闭off

## Mybatis

### 程序的层次结构

- @Controller   @RestController
- @Service
- @Repository
- @Component

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/springboot/1.png?raw=true)

功能是相似的，根据层次来区分。对应三个层次，增加程序的可读性。

@Component是个例外，如果多个层次都用就使用这个注解。

分包可以按照功能划分或者按照层次划分

### 分层之间的传输

- PO = Persistant Object 持久化对象   		数据访问层
- DTO = Data Transfer Object 数据传输对象      最外层通信
- VO = Value Object 或View Object        显示对象，在mvc中传递
- POJO = Pure Old Java Object /Plain Ordinary Java Object      一种比较单纯的Java对象
- DO = Domain Object ; BO = Business Object 处理业务逻辑    少见，同一种东西
- DAO  = Data Access Object      数据访问对象，处理和数据库发生交互的，增删改查都是通过DAO操作

POJO和javaBean

POJO：所有属性都是私有的，只通过get和set开放出来

JavaBean：会有一些业务逻辑



### 添加Mybatis

- 修改pom.xml ，添加mybatis支持
- 修改application.yml添加数据库连接
- 修改启动类，增加@MapperScan("package-of-mapping")注解
- 添加Mybatis Mapping接口
- 添加Mapping 对应的XML(可选)
- 在对应接口上添加注解@Mapper

## Spring Boot 单元测试

Assert   -Junit的断言

- 判断某条件是否为真Assert.assertTrue(条件表达式);

- 判断某条件是否为假Assert.assertFalse(条件表达式);

- 判断两个变量值是否相同Assert.assertEquals(v1,v2);

- 判断两个变量值是否不相同Assert.assertNotEquals(v1,v2);

- 判断两个数组是否相同Assert.assertArrayEquals(数组1，数组2);

- 直接测试失败Assert.fail()  Assert.fail(message)

  ### Assert    vs  assert

- Assert是JUnit的断言类，全名是org.junit.Assert

- Assert提供了很多静态方法，例如assertTrue,assertFalse,assertNotNull,assertNull,assertEquals,assertNotEquals等

- assert是java关键字，使用方法有两种，表达式为false时，jvm会退出

  - assert表达式；	assert表达式：表达式不成立后的提示信息

- assert关键字内表达式是否被检查成立依赖jvm的参数，默认是关闭的

### 概念

- 驱动模块
- 被测模块
- 桩模块

场景1:测试A模块，但是要传入进来的桩模块还没有开发完。模拟桩模块

场景2：数据访问量比较大，代替数据访问层，检查是否正确。

Aseert示例1：

```java
    public static double calMonthSalary(int workdays,double monthSalary){
        return monthSalary/21.75*workdays;
    }
```

月薪2175，工作了一天，使用Asset查看结果是否等于100

```java
Assert.assertTrue(calMonthSalary(1,2175)==100);
```

Aseert示例2：

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/springboot/%E8%A3%85%E6%A8%A1%E5%9D%97%E7%A4%BA%E4%BE%8B.png?raw=true)

### Mockito

首先加入依赖

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <scope>test</scope>
</dependency>
```

**TDD**

- Test Driven Development（测试驱动开发） 
- 先写测试用例，后写实现代码
- 重构现有代码时特别好用

**RDD**

Resume Driven Devolopment 简历驱动开发

#### 使用mockito做桩模块来测试业务逻辑层



## SpringBoot 应用监控

### 遇到的问题

1. 不显示详细信息
2. 只有health和info没有其他的接口可以访问

### 操作

1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2. 配置application.yml

```xml
management:
  endpoints:
    web:
      exposure:
        include: '*'
//配置之后会显示全部的端点，否则只显示health和info
   endpoint:
    health:
      show-details: always
//配置后显示detail信息
```



![](https://user-images.githubusercontent.com/24977343/53216720-7bb28b80-3690-11e9-8bb8-a2238fc9da65.png)

这个图是之前的版本，2.0的已经做了一些修改。仅供参考

### 定制端点

1. 修改端点id		//TODO
2. 开启端点

```xml
management:
  endpoints:
	enabled-by-default: true
```

3. 关闭端点	将enabled-by-default: true    改为false

4. 只开启所需端点

   - 关闭所有端点      enabled-by-default: false
   - 开启所需端口例如health

   ```xml
   management:
   	endpoint:
   		health:
   			enabled: true
   ```

5. 自定义端点访问端口

```xml
management:
	  server:
    	port: 8081
```

重启后访问地址为：

http://localhost:8081/actuator

6. 关闭端口    将端口号改为-1

### 自定义端点

​	当Spring Boot提供的端点不能满足我们的特殊需求，而我们又需要对特殊的应用状态进行监控的时候，就需要自定义一个端点。

## Spring Boot开发Web应用

**默认配置**

Spring Boot默认提供静态资源目录位置需要于classpath下，目录名需符合如下规则：

- /static
- /public
- resources
- /META-INF/resources

### Spring Boot提供了默认配置的模板引擎主要有以下几种：

- Thymeleaf
- FreeMarker
- Velocity
- Groovy
- Mustache

**Spring Boot建议使用这些模板引擎，避免使用JSP，若一定要使用JSP将无法实现Spring Boot的多种特性**

​	当你使用上述模板引擎中的任何一个，它们默认的模板配置路径为：`src/main/resources/templates`。当然也可以修改这个路径，具体如何修改，可在后续各模板引擎的配置属性中查询并修改。

#### Thymeleaf

​	Thymeleaf是一个XML/XHTML/HTML5模板引擎，可用于Web与非Web环境中的应用开发。它是一个开源的Java库，基于Apache License 2.0许可，由Daniel Fernández创建，该作者还是Java加密库Jasypt的作者。

​	Thymeleaf提供了一个用于整合Spring MVC的可选模块，在应用开发中，你可以使用Thymeleaf来完全代替JSP或其他模板引擎，如Velocity、FreeMarker等。Thymeleaf的主要目标在于提供一种可被浏览器正确显示的、格式良好的模板创建方式，因此也可以用作静态建模。你可以使用它创建经过验证的XML与HTML模板。相对于编写逻辑或代码，开发者只需将标签属性添加到模板中即可。接下来，这些标签属性就会在DOM（文档对象模型）上执行预先制定好的逻辑。

```jsp
<table>
  <thead>
    <tr>
      <th th:text="#{msgs.headers.name}">Name</td>
      <th th:text="#{msgs.headers.price}">Price</td>
    </tr>
  </thead>
  <tbody>
    <tr th:each="prod : ${allProducts}">
      <td th:text="${prod.name}">Oranges</td>
      <td th:text="${#numbers.formatDecimal(prod.price,1,2)}">0.99</td>
    </tr>
  </tbody>
</table>
```

​	可以看到Thymeleaf主要以属性的方式加入到html标签中，浏览器在解析html时，当检查到没有的属性时候会忽略，所以Thymeleaf的模板可以通过浏览器直接打开展现，这样非常有利于前后端的分离。

​	在Spring Boot中使用Thymeleaf，只需要引入下面依赖，并在默认的模板路径`src/main/resources/templates`下编写模板文件即可完成。

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

在完成配置之后，举一个简单的例子，在快速入门工程的基础上，举一个简单的示例来通过Thymeleaf渲染一个页面。

```java
@Controller
public class HelloController {
    
    @ResponseBody
    @RequestMapping("/hello")
public String hello() {
    return "Hello World";
}

    @RequestMapping("/")
    public String index(ModelMap map) {
        map.addAttribute("host", "http://blog.didispace.com");
        return "index";
    }

}
```

​		

```html
<!DOCTYPE html>
<html xmlns:th="http://www.w3.org/1999/xhtml">
<head lang="en">
    <meta charset="UTF-8" />
    <title></title>
</head>
<body>
<h1 th:text="${host}">Hello World</h1>
</body>
</html>
```

如上页面，直接打开html页面展现Hello World，但是启动程序后，访问`http://localhost:8080/`，则是展示Controller中host的值：`http://blog.didispace.com`，做到了不破坏HTML自身内容的数据逻辑分离。

这里返回的index其实应该是一个路径，然后跳转过去，并带着参数。在index.html中通过表达式获取参数

```html
<h1 th:text="${host}">Hello World</h1>
```

如有需要修改默认配置的时候，只需复制下面要修改的属性到`application.properties`中，并修改成需要的值，如修改模板文件的扩展名，修改默认的模板路径等。

```properties
# Enable template caching.
spring.thymeleaf.cache=true 
# Check that the templates location exists.
spring.thymeleaf.check-template-location=true 
# Content-Type value.
spring.thymeleaf.content-type=text/html 
# Enable MVC Thymeleaf view resolution.
spring.thymeleaf.enabled=true 
# Template encoding.
spring.thymeleaf.encoding=UTF-8 
# Comma-separated list of view names that should be excluded from resolution.
spring.thymeleaf.excluded-view-names= 
# Template mode to be applied to templates. See also StandardTemplateModeHandlers.
spring.thymeleaf.mode=HTML5 
# Prefix that gets prepended to view names when building a URL.
spring.thymeleaf.prefix=classpath:/templates/ 
# Suffix that gets appended to view names when building a URL.
spring.thymeleaf.suffix=.html  spring.thymeleaf.template-resolver-order= 
```

#### FreeMarker

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8" />
    <title></title>
</head>
<body>
FreeMarker模板引擎
<h1>${host}</h1>
</body>
</html>
```

其余部分都一样。文件结尾ftl

#### Velocity

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8" />
    <title></title>
</head>
<body>
Velocity模板
<h1>${host}</h1>
</body>
</html>
```

其余部分一样，文件结尾为vm

### Spring Boot中使用Swagger2构建强大的RESTful API文档

Swagger2，它可以轻松的整合到Spring Boot中，并与Spring MVC程序配合组织出强大RESTful API文档。它既可以减少我们创建文档的工作量，同时说明内容又整合入实现代码中，让维护文档和修改代码整合为一体，可以让我们在修改代码逻辑的同时方便的修改文档说明。另外Swagger2也提供了强大的页面测试功能来调试每个RESTful API。具体效果如下图所示：

![](http://blog.didispace.com/content/images/2016/04/swagger2_1.png)

如果在Spring Boot中使用Swagger2。首先，我们需要一个Spring Boot实现的RESTful API工程

**添加Swagger2依赖**

```xml

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.2.2</version>
</dependency>
```

**创建Swagger2配置类**

在`Application.java`同级创建Swagger2的配置类`Swagger2`。

### 设置统一的异常处理

## Spring Boot事务

### 什么是事务？

对于业务人员的一个操作实际是对数据读写的多步操作的结合。由于数据操作在顺序执行的过程中，任何一步操作都有可能发生异常，异常会导致后续操作无法完成，此时由于业务逻辑并未正确的完成，之前成功操作数据的不可靠，需要在这种情况下进行回退。

事务的作用就是保证用户的每一步操作都是可靠的，事务中的每一步操作都必须成功执行，只要发生异常就回退到事务开始未进行操作的状态。

### 快速入门

在Spring Boot中，当我们使用了spring-boot-starter-jdbc或spring-boot-starter-data-jpa依赖的时候，框架会自动默认分别注入DataSourceTransactionManager或JpaTransactionManager。所以我们不需要任何额外配置就可以用@Transactional注解进行事务的使用。

## Spring Boot缓存

**引入缓存**

- 在`pom.xml`中引入cache依赖，添加如下内容：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

- 在Spring Boot主类中增加`@EnableCaching`注解开启缓存功能，如下：

```java
@SpringBootApplication
@EnableCaching
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

- 在数据访问接口中，增加缓存配置注解，如：

```java
@CacheConfig(cacheNames = "users")
public interface UserRepository extends JpaRepository<User, Long> {

    @Cacheable
    User findByName(String name);

}
```

## Cache注解详解

回过头来我们再来看，这里使用到的两个注解分别作了什么事情。

- `@CacheConfig`：主要用于配置该类中会用到的一些共用的缓存配置。在这里`@CacheConfig(cacheNames = "users")`：配置了该数据访问对象中返回的内容将存储于名为users的缓存对象中，我们也可以不使用该注解，直接通过`@Cacheable`自己配置缓存集的名字来定义。
- 















































<h1 th:text="${host}">

