---
title: Spring 注解和配置
tags: Spring
categories: 框架
---
### Spring-XML

#### `<context:component-scan base-package="" >`

```xml
    <!--自动扫描base-package 对应路径或者改路径子包下面的java文件，带有@Service、@Controller等这些注解的类，则把这些类注册为bean-->
    <context:component-scan base-package="com">
        <!--只想扫描包下面的 controller可以用  include-filter-->
        <context:include-filter type="assignable" expression="com" />
        <context:exclude-filter type="assignable" expression="com" />
    </context:component-scan>
```

`context:component-scan`

自动扫描base-package 对应路径或者改路径子包下面的java文件。将带有@Service、@Controller等这些注解的类，则把这些类注册为bean。

`<context:include-filter` ：只扫描这个下面的包，并起作用

这个需要与 `use-dafault-filters=”false”`搭配使用，否则不起作用。

`<context:exclude-filter`：除这个路径之外的起作用

| Filter Type |    Examples Expression     |                         Description                          |
| :---------: | :------------------------: | :----------------------------------------------------------: |
| annotation  | org.example.SomeAnnotation |               符合SomeAnnoation的target class                |
| assignable  |   org.example.SomeClass    |                  指定class或interface的全名                  |
|   aspectj   |   org.example..*Service+   |                          AspetJ语法                          |
|    regex    |   org.example.Default.*    |                      Regelar Expression                      |
|   custom    |  org.example.MyTypeFilter  | Spring3新增自订Type,称作org.springframework.core.type.TypeFilter |

#### `<mvc:annotation-driven>`

​		`<`mvc:annotation-driven`>`会自动注册`RequestMappingHandlerMapping`与`RequestMappingHandlerAdapter`两个Bean,这是Spring MVC为@Controller分发请求所必需的，并且提供了数据绑定支持，`@NumberFormatannotation`支持，`@DateTimeFormat`支持,@Valid支持读写XML的支持（JAXB）和读写JSON的支持（默认Jackson）等功能

​	

```xml
<mvc:annotation-driven>
   <mvc:message-converters register-defaults="true">
      <!-- 配置Fastjson支持 -->
      <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
         <property name="supportedMediaTypes">
            <list>
               <value>application/json</value>
            </list>
         </property>
         <property name="features">
            <value>WriteDateUseDateFormat</value>
            <!--<list>-->
            <!--<value>WriteDateUseDateFormat</value>-->
            <!--<value>QuoteFieldNames</value>-->
            <!--<value>WriteNullListAsEmpty</value>-->
            <!--<value>WriteNullStringAsEmpty</value>-->
            <!--<value>WriteNullNumberAsZero</value>-->
            <!--<value>WriteNullBooleanAsFalse</value>-->
            <!--<value>WriteMapNullValue</value>-->
            <!--</list>-->
         </property>
      </bean>
   </mvc:message-converters>
</mvc:annotation-driven>
```

**<mvc:message-converters register-defaults="true">**	

​	在说 `<mvc:message-converters register-defaults="true">`之前要说一下

**`@ResponseBody注解`**

​	很明显这个注解是将方法的返回值作为response的body部分。首先就是方法返回的类型，可以是字节、数组、字符串、对象引用等，将这些返回类型以什么样的内容格式(即response的content-type类型，同时还要考虑到客户是否接受这个类型)存进response的body中返回给客户端是一个问题，对于	这个过程的处理都是靠许许多多的HttpMessageConverter转换器来完成的。

​	常用的content-type类型有：text/html、text/plain、text/xml、application/json、application/x-www-form-urlencoded、image/png等，不同的类型，对body中的数据的解析也是不一样的。 

​	<mvc:message-converters register-defaults="true">有一个register-defaults属性，当为true时，仍然注册默认的HttpMessageConverter，当为false则不注册，仅仅使用用户自定义的HttpMessageConverter。 	

​	HttpMessageConverter主要针对那些不会返回view视图的response： 

含有方法含有**@ResponseBody**或者返回值为**HttpEntity**等类型的，它们都会用到HttpMessageConverter。以@ResponseBody举例：

首先先决定由那个HandlerMethodReturnValueHandler来处理返回值，由于是@ResponseBody所以将会由RequestResponseBodyMethodProcessor来处理，选取一个合适的content-type，再由这个content-type和返回类型来选取合适的HttpMessageConverter，找到合适的HttpMessageConverter后，便调用它的write方法。 

这里是fastjson

#### `<mvc:default-servlet-handler />`

​	如果将DispatcherServlet请求映射配置为"/"，则Spring MVC将捕获Web容器所有的请求，包括静态资源的请求，Spring MVC会将它们当成一个普通请求处理，因此找不到对应处理器将导致错误。

​	在springMVC-servlet.xml中配置<mvc:default-servlet-handler />后，会在Spring MVC上下文中定义一个org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。

#### `mvc:view-controller`	

```xml
<mvc:view-controller path="/" view-name="redirect:${shiro.loginUrl}" />
```

在springmvc中使用mvc:view-controller标签直接将访问url和视图进行映射，而无需要通过控制器。

