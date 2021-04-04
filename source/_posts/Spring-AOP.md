---
title: Spring--AOP
date: 2018-05-24 16:21:43
tags: Spring
categories: 框架
---

![](https://github.com/whyathere/tuchuang/blob/master/%E7%BE%8E%E5%9B%BE/4.jpg?raw=true)

## AOP基础

### AOP的基本概念

​       在进行AOP开发前，先熟悉几个概念：

   

- **连接点（Jointpoint）：**表示需要在程序中插入横切关注点的扩展点，连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等，Spring只支持方法执行连接点，**在AOP中表示为“在哪里干”**；
- **切入点（Pointcut）：**选择一组相关连接点的模式，即可以认为连接点的集合，Spring支持perl5正则表达式和AspectJ切入点模式，Spring默认使用AspectJ语法，**在AOP中表示为“在哪里干的集合”**；
- **通知（Advice）：**在连接点上执行的行为，通知提供了在AOP中需要在切入点所选择的连接点处进行扩展现有行为的手段；包括前置通知（before advice）、后置通知(after advice)、环绕通知（around advice），在Spring中通过代理模式实现AOP，并通过拦截器模式以环绕连接点的拦截器链织入通知；**在AOP中表示为“干什么”；**
- **方面/切面（Aspect）：**横切关注点的模块化，比如上边提到的日志组件。可以认为是通知、引入和切入点的组合；在Spring中可以使用Schema和@AspectJ方式进行组织实现；**在AOP中表示为“在哪干和干什么集合”；**
- **引入（inter-type declaration）：**也称为内部类型声明，为已有的类添加额外新的字段或方法，Spring允许引入新的接口（必须对应一个实现）到所有被代理对象（目标对象）, **在AOP中表示为“干什么（引入什么）”**；
- **目标对象（Target Object）：**需要被织入横切关注点的对象，即该对象是切入点选择的对象，需要被通知的对象，从而也可称为“被通知对象”；由于Spring AOP 通过代理模式实现，从而这个对象永远是被代理对象，**在AOP中表示为“对谁干”**；
- **AOP代理（AOP Proxy）：**AOP框架使用代理模式创建的对象，从而实现在连接点处插入通知（即应用切面），就是**通过代理来对目标对象应用切面**。在Spring中，AOP代理可以用JDK动态代理或CGLIB代理实现，而通过拦截器模型应用切面。
- **织入（Weaving）：**织入是一个过程，是将切面应用到目标对象从而创建出AOP代理对象的过程，织入可以在编译期、类装载期、运行期进行。

 在AOP中，通过切入点选择目标对象的连接点，然后在目标对象的相应连接点处织入通知，而切入点和通知就是切面（横切关注点），而在目标对象连接点处应用切面的实现方式是通过AOP代理对象，如图6-2所示。

 ![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/Spring/AOP%E6%A6%82%E5%BF%B5%E5%85%B3%E7%B3%BB.JPG?raw=true)

图6-2 概念关系

 接下来再让我们具体看看Spring有哪些通知类型：

- **前置通知（Before Advice）:**在切入点选择的连接点处的方法之前执行的通知，该通知不影响正常程序执行流程（除非该通知抛出异常，该异常将中断当前方法链的执行而返回）。
- **后置通知（After Advice）:**在切入点选择的连接点处的方法之后执行的通知，包括如下类型的后置通知：
- **后置返回通知(After returning Advice)：**在切入点选择的连接点处的方法抛出异常返回时执行的通知，必须是连接点处的方法抛出任何异常返回时才调用异常通知。
- **后置异常通知(After throwing Advice):**在切入点选择的连接点处的方法抛出异常返回时执行的通知，必须是连接点处的方法抛出任何异常返回时才调用异常通知。
- **后置最终通知(After finally Advice):**在切入点选择的连接点处的方法返回时执行的通知，不管抛没抛出异常都执行，类似于Java的finally块。
- **环绕通知(Around Advice):**环绕着在切入点选择的连接点处的方法所执行的通知，环绕通知可以在方法调用之前和之后自定义任何行为，并且可以决定是否执行连接点处的方法、替换返回值、抛出异常等等。

各种通知类型在UML序列图中的位置如图6-3所示:
   ![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/Spring/%E9%80%9A%E7%9F%A5%E7%B1%BB%E5%9E%8B.JPG?raw=true)

图6-3 通知类型

##  AOP代理

### Demo

​       AOP代理就是AOP框架通过代理模式创建的对象，Spring使用JDK动态代理或CGLIB代理来实现，Spring缺省使用JDK动态代理来实现，从而任何接口都可别代理，如果被代理的对象实现不是接口将默认使用CGLIB代理，不过CGLIB代理当然也可应用到接口。

​        **AOP代理的目的就是将切面织入到目标对象。**

​        概念都将完了，接下来让我们看一下AOP的 HelloWorld!吧。

#### 准备环境 

​       首先准备开发需要的jar包，请到spring-framework-3.0.5.RELEASE-dependencies.zip和spring-framework-3.0.5.RELEASE-with-docs中查找如下jar包：

  org.springframework.aop-3.0.5.RELEASE.jar

  com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar

  com.springsource.org.aopalliance-1.0.0.jar

   com.springsource.net.sf.cglib-2.2.0.jar 

#### 定义目标类

​       1）定义目标接口：

 ```java
public interface IHelloWorldService {
    public void sayHello();
}
 ```

​       2）定义目标接口实现：

 ```java
public class HelloWorldService implements IHelloWorldService {
    public void sayHello() {
        System.out.println("============Hello World!");
    }
}
 ```

​       注：在日常开发中最后将业务逻辑定义在一个专门的service包下，而实现定义在service包下的impl包中，服务接口以IXXXService形式，而服务实现就是XXXService，这就是规约设计，见名知义。当然可以使用公司内部更好的形式，只要大家都好理解就可以了。

####   定义切面支持类

​       有了目标类，该定义切面了，切面就是通知和切入点的组合，而切面是通过配置方式定义的，因此这定义切面前，我们需要定义切面支持类，切面支持类提供了通知实现：

 ```java
public class HelloWorldAspect {
    //前置通知
    public void beforeAdvice() {
        System.out.println("===========before advice");
    }
    //后置最终通知
    public void afterFinallyAdvice() {
        System.out.println("===========after finally advice");
    }
}
 ```

​       此处HelloWorldAspect类不是真正的切面实现，只是定义了通知实现的类，在此我们可以把它看作就是缺少了切入点的切面。

​        注：对于AOP相关类最后专门放到一个包下，如“aop”包，因为AOP是动态织入的，所以如果某个目标类被AOP拦截了并应用了通知，可能很难发现这个通知实现在哪个包里，因此推荐使用规约命名，方便以后维护人员查找相应的AOP实现。

####  在XML中进行配置

有了通知实现，那就让我们来配置切面吧：

​       1）首先配置AOP需要aop命名空间，配置头如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans  xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
    
</beans>
```

​       2）配置目标类：

 ```xml
 <bean id="helloWorldService" class="com.zero.aop.HelloWorldService"/>
 ```

​       3）配置切面：

 ```java
    <bean id="helloWorldService" class="com.zero.aop.HelloWorldService"/>

    <bean id="aspect" class="com.zero.aop.HelloWorldAspect"/>
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* com.zero.aop.*.*(..))"/>
        <aop:aspect ref="aspect">
            <aop:before method="beforeAdvice" pointcut-ref="pointcut"/>
            <aop:after pointcut="execution(* com.zero.aop.*.*(..))" method="afterFinallyAdvice"/>
        </aop:aspect>
    </aop:config>
 ```

​       切入点使用<aop:config>标签下的<aop:pointcut>配置，expression属性用于定义切入点模式，默认是AspectJ语法，“execution(* cn.javass..*.*(..))”表示匹配cn.javass包及子包下的任何方法执行。

 切面使用<aop:config>标签下的<aop:aspect>标签配置，其中“ref”用来引用切面支持类的方法。

 前置通知使用<aop:aspect>标签下的<aop:before>标签来定义，pointcut-ref属性用于引用切入点Bean，而method用来引用切面通知实现类中的方法，该方法就是通知实现，即在目标类方法执行之前调用的方法。

 最终通知使用<aop:aspect>标签下的<aop:after >标签来定义，切入点除了使用pointcut-ref属性来引用已经存在的切入点，也可以使用pointcut属性来定义，如pointcut="execution(* cn.javass..*.*(..))"，method属性同样是指定通知实现，即在目标类方法执行之后调用的方法。

####   运行测试

测试类非常简单，调用被代理Bean跟调用普通Bean完全一样，Spring AOP将为目标对象创建AOP代理，具体测试代码如下：

 ```java
public class AopTest {
    @Test
    public void testHelloworld() {
        ApplicationContext ctx =  new ClassPathXmlApplicationContext("aop.xml");
        IHelloWorldService helloworldService =
                ctx.getBean("helloWorldService", IHelloWorldService.class);
        helloworldService.sayHello();
    }
}
 ```

​       该测试将输出如下如下内容：

 ===========before advice
============Hello World!
===========after finally advice

​        从输出我们可以看出：前置通知在切入点选择的连接点（方法）之前允许，而后置通知将在连接点（方法）之后执行，具体生成AOP代理及执行过程如图6-4所示。

 ![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/Spring/Spring%20AOP%E6%A1%86%E6%9E%B6%E7%94%9F%E6%88%90AOP%E4%BB%A3%E7%90%86%E8%BF%87%E7%A8%8B.JPG?raw=true)

 图6-4 Spring AOP框架生成AOP代理过程

###  基于Schema的AOP

#### 基于Schema的AOP

​     基于Schema的AOP从Spring2.0之后通过“aop”命名空间来定义切面、切入点及声明通知。

 在Spring配置文件中，所有AOP相关定义必须放在<aop:config>标签下，该标签下可以有<aop:pointcut>、<aop:advisor>、<aop:aspect>标签，配置顺序不可变。 

- <aop:pointcut>：用来定义切入点，该切入点可以重用；
- <aop:advisor>：用来定义只有一个通知和一个切入点的切面；
- <aop:aspect>：用来定义切面，该切面可以包含多个切入点和通知，而且标签内部的通知和切入点定义是无序的；和advisor的区别就在此，advisor只包含一个通知和一个切入点。

 ![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/Spring/aop-schema.JPG?raw=true)

#### 声明切面

​    切面就是包含切入点和通知的对象，在Spring容器中将被定义为一个Bean，Schema方式的切面需要一个切面支持Bean，该支持Bean的字段和方法提供了切面的状态和行为信息，并通过配置方式来指定切入点和通知实现。

​      切面使用<aop:aspect>标签指定，ref属性用来引用切面支持Bean。

​     切面支持Bean“aspectSupportBean”跟普通Bean完全一样使用，切面使用“ref”属性引用它。

####   声明切入点

​    切入点在Spring中也是一个Bean，Bean定义方式可以有很三种方式：

​     **1）在<aop:config>标签下使用<aop:pointcut>声明一个切入点Bean**，该切入点可以被多个切面使用，对于需要共享使用的切入点最好使用该方式，该切入点使用id属性指定Bean名字，在通知定义时使用pointcut-ref属性通过该id引用切入点，expression属性指定切入点表达式：

 ```xml
<aop:config>  
 <aop:pointcut id="pointcut" expression="execution(* cn.javass..*.*(..))"/>  
 <aop:aspect ref="aspectSupportBean">  
    <aop:before pointcut-ref="pointcut" method="before"/>  
 </aop:aspect>  
</aop:config>  
 ```

    **2）在<aop:aspect>*标签下使用<aop:pointcut>声明一个切入点Bean**，该切入点可以被多个切面使用，但一般该切入点只被该切面使用，当然也可以被其他切面使用，但最好不要那样使用，该切入点使用id属性指定Bean名字，在通知定义时使用pointcut-ref属性通过该id引用切入点，expression属性指定切入点表达式： 

 ```xml
<aop:config>  
 <aop:aspect ref="aspectSupportBean">  
    <aop:pointcut id=" pointcut" expression="execution(* cn.javass..*.*(..))"/>  
    <aop:before pointcut-ref="pointcut" method="before"/>  
 </aop:aspect>  
</aop:config>  
 ```

​    **3）匿名切入点Bean，**可以在声明通知时通过pointcut属性指定切入点表达式，该切入点是匿名切入点，只被该通知使用：

 ```xml
<aop:config>  
 <aop:aspect ref="aspectSupportBean">  
     <aop:after pointcut="execution(* cn.javass..*.*(..))" method="afterFinallyAdvice"/> 
 </aop:aspect>  
</aop:config>  
 ```

#### 声明通知

 基于Schema方式支持前边介绍的5中通知类型：

 **一、前置通知：**在切入点选择的方法之前执行，通过<aop:aspect>标签下的<aop:before>标签声明：

 ```xml
<aop:before pointcut="切入点表达式"  pointcut-ref="切入点Bean引用"  
method="前置通知实现方法名"  
arg-names="前置通知实现方法参数列表参数名字"/>  
 ```

 **pointcut和pointcut-ref：**二者选一，指定切入点；

​         **method：**指定前置通知实现方法名，如果是多态需要加上参数类型，多个用“，”隔开，如beforeAdvice(java.lang.String)；

​         **arg-names：**指定通知实现方法的参数名字，多个用“，”分隔，可选，类似于【3.1.2 构造器注入】中的参数名注入限制：**在class文件中没生成变量调试信息是获取不到方法参数名字的，因此只有在类没生成变量调试信息时才需要使用arg-names属性来指定参数名，如**arg-names="param"表示通知实现方法的参数列表的第一个参数名字为“param”。

首先在cn.javass.spring.chapter6.service.IhelloWorldService定义一个测试方法：

 ```java
public void sayBefore(String param);  
 ```

其次在cn.javass.spring.chapter6.service.impl. HelloWorldService定义实现

 ```java
@Override  
public void sayBefore(String param) {  
    System.out.println("============say " + param);  
}  
 ```

第三在cn.javass.spring.chapter6.aop. HelloWorldAspect定义通知实现：

 ```java
public void beforeAdvice(String param) {  
    System.out.println("===========before advice param:" + param);  
}  
 ```

最后在chapter6/advice.xml配置文件中进行如下配置：

 ```xml
<bean id="helloWorldService" class="cn.javass.spring.chapter6.service.impl.HelloWorldService"/>  
<bean id="aspect" class="cn.javass.spring.chapter6.aop.HelloWorldAspect"/>  
<aop:config>  
    <aop:aspect ref="aspect">  
        <aop:before pointcut="execution(* cn.javass..*.sayBefore(..)) and args(param)"   
                           method="beforeAdvice(java.lang.String)"   
                           arg-names="param"/>  
    </aop:aspect>  
</aop:config>
 ```

测试代码cn.javass.spring.chapter6.AopTest:

 ```java
@Test  
public void testSchemaBeforeAdvice(){  
     System.out.println("======================================");  
     ApplicationContext ctx = new ClassPathXmlApplicationContext("chapter6/advice.xml");  
     IHelloWorldService helloworldService = ctx.getBean("helloWorldService", IHelloWorldService.class);  
     helloworldService.sayBefore("before");  
    System.out.println("======================================");  
}  
 ```

将输入：

|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | =====================================================before advice param:before============say before========================================== |

 

 

 

 

 

 

 

 

分析一下吧：

**1）切入点匹配：**在配置中使用“execution(* cn.javass..*.sayBefore(..)) ”匹配目标方法sayBefore，且使用“args(param)”匹配目标方法只有一个参数且传入的参数类型为通知实现方法中同名的参数类型；

**2）目标方法定义：**使用method=" beforeAdvice(java.lang.String) "指定前置通知实现方法，且该通知有一个参数类型为java.lang.String参数；

**3）目标方法参数命名：**其中使用arg-names=" param "指定通知实现方法参数名为“param”，切入点中使用“args(param)”匹配的目标方法参数将自动传递给通知实现方法同名参数。

   **二、后置返回通知：**在切入点选择的方法正常返回时执行，通过<aop:aspect>标签下的<aop:after-returning>标签声明：

 ```xml
<aop:after-returning pointcut="切入点表达式"  pointcut-ref="切入点Bean引用"  
    method="后置返回通知实现方法名"  
    arg-names="后置返回通知实现方法参数列表参数名字"  
    returning="返回值对应的后置返回通知实现方法参数名"  
/>  
 ```

 **pointcut和pointcut-ref：**同前置通知同义；

​         **method：**同前置通知同义；

​         **arg-names：**同前置通知同义；

​         **returning：**定义一个名字，该名字用于匹配通知实现方法的一个参数名，当目标方法执行正常返回后，将把目标方法返回值传给通知方法；returning限定了只有目标方法返回值匹配与通知方法相应参数类型时才能执行后置返回通知，否则不执行，对于returning对应的通知方法参数为Object类型将匹配任何目标返回值。

首先在cn.javass.spring.chapter6.service.IhelloWorldService定义一个测试方法：

 ```java
public boolean sayAfterReturning();  
 ```

其次在cn.javass.spring.chapter6.service.impl. HelloWorldService定义实现

 ```java
@Override  
public boolean sayAfterReturning() {  
    System.out.println("============after returning");  
    return true;  
}  
 ```

第三在cn.javass.spring.chapter6.aop. HelloWorldAspect定义通知实现：

 ```java
public void afterReturningAdvice(Object retVal) {  
    System.out.println("===========after returning advice retVal:" + retVal);  
}  
 ```

最后在chapter6/advice.xml配置文件中接着前置通知配置的例子添加如下配置：

 ```xml
<aop:after-returning pointcut="execution(* cn.javass..*.sayAfterReturning(..))"  
                                method="afterReturningAdvice"  
                               arg-names="retVal"    
                               returning="retVal"/>  
 ```

测试代码cn.javass.spring.chapter6.AopTest:

 ```java
@Test  
public void testSchemaAfterReturningAdvice() {  
    System.out.println("======================================");  
    ApplicationContext ctx = new ClassPathXmlApplicationContext("chapter6/advice.xml");  
    IHelloWorldService helloworldService = ctx.getBean("helloWorldService", IHelloWorldService.class);  
    helloworldService.sayAfterReturning();      
    System.out.println("======================================");  
} 
 ```

将输入：

|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ==================================================after returning===========after returning advice retVal:true====================================== |

 

 

 

 

 

 

 

 

分析一下吧：

**1）切入点匹配：**在配置中使用“execution(* cn.javass..*.sayAfterReturning(..)) ”匹配目标方法sayAfterReturning，该方法返回true；

**2）目标方法定义：**使用method="afterReturningAdvice"指定后置返回通知实现方法；

**3）目标方法参数命名：**其中使用arg-names="retVal"指定通知实现方法参数名为“retVal”；

**4）返回值命名：**returning="retVal"用于将目标返回值赋值给通知实现方法参数名为“retVal”的参数上。

 **三、后置异常通知：**在切入点选择的方法抛出异常时执行，通过<aop:aspect>标签下的<aop:after-throwing>标签声明：

 ```xml
<aop:after-throwing pointcut="切入点表达式"  pointcut-ref="切入点Bean引用"  
                                method="后置异常通知实现方法名"  
                                arg-names="后置异常通知实现方法参数列表参数名字"  
                                throwing="将抛出的异常赋值给的通知实现方法参数名"/>  
 ```

   **pointcut和pointcut-ref：**同前置通知同义；

​         **method：**同前置通知同义；

​         **arg-names：**同前置通知同义；

​         **throwing：**定义一个名字，该名字用于匹配通知实现方法的一个参数名，当目标方法抛出异常返回后，将把目标方法抛出的异常传给通知方法；throwing限定了只有目标方法抛出的异常匹配与通知方法相应参数异常类型时才能执行后置异常通知，否则不执行，对于throwing对应的通知方法参数为Throwable类型将匹配任何异常。

 

首先在cn.javass.spring.chapter6.service.IhelloWorldService定义一个测试方法：

 ```java
public void sayAfterThrowing();  
 ```

其次在cn.javass.spring.chapter6.service.impl. HelloWorldService定义实现

 ```java
@Override  
public void sayAfterThrowing() {  
    System.out.println("============before throwing");  
    throw new RuntimeException();  
} 
 ```

第三在cn.javass.spring.chapter6.aop. HelloWorldAspect定义通知实现：

 ```java
public void afterThrowingAdvice(Exception exception) {  
  System.out.println("===========after throwing advice exception:" + exception);  
}  
 ```

最后在chapter6/advice.xml配置文件中接着前置通知配置的例子添加如下配置：

 ```xml
<aop:after-throwing pointcut="execution(* cn.javass..*.sayAfterThrowing(..))"  
                                method="afterThrowingAdvice"  
                                arg-names="exception"  
                                throwing="exception"/>  
 ```

测试代码cn.javass.spring.chapter6.AopTest:

 ```java
@Test(expected = RuntimeException.class)  
public void testSchemaAfterThrowingAdvice() {  
    System.out.println("======================================");  
    ApplicationContext ctx = new ClassPathXmlApplicationContext("chapter6/advice.xml");  
    IHelloWorldService helloworldService = ctx.getBean("helloWorldService", IHelloWorldService.class);  
    helloworldService.sayAfterThrowing();  
    System.out.println("======================================");  
}  
 ```

将输入：

|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ==================================================before throwing===========after throwing advice exception:java.lang.RuntimeException====================================== |

 分析一下吧：

**1）切入点匹配：**在配置中使用“execution(* cn.javass..*.sayAfterThrowing(..))”匹配目标方法sayAfterThrowing，该方法将抛出RuntimeException异常；

**2）目标方法定义：**使用method="afterThrowingAdvice"指定后置异常通知实现方法；

**3）目标方法参数命名：**其中使用arg-names="exception"指定通知实现方法参数名为“exception”；

**4）异常命名：**returning="exception"用于将目标方法抛出的异常赋值给通知实现方法参数名为“exception”的参数上。

 

 **四、后置最终通知：**在切入点选择的方法返回时执行，不管是正常返回还是抛出异常都执行，通过<aop:aspect>标签下的<aop:after >标签声明：

 ```xml
<aop:after pointcut="切入点表达式"  pointcut-ref="切入点Bean引用"  
                  method="后置最终通知实现方法名"  
                  arg-names="后置最终通知实现方法参数列表参数名字"/> 
 ```

​         **pointcut和pointcut-ref：**同前置通知同义；

​         **method：**同前置通知同义；

​         **arg-names：**同前置通知同义；

 首先在cn.javass.spring.chapter6.service.IhelloWorldService定义一个测试方法：

 ```java
public boolean sayAfterFinally();  
 ```

其次在cn.javass.spring.chapter6.service.impl. HelloWorldService定义实现

 分析一下吧：

**1）切入点匹配：**在配置中使用“execution(* cn.javass..*.sayAfterFinally(..))”匹配目标方法sayAfterFinally，该方法将抛出RuntimeException异常；

**2）目标方法定义：**使用method=" afterFinallyAdvice "指定后置最终通知实现方法。

 **五、环绕通知：**环绕着在切入点选择的连接点处的方法所执行的通知，环绕通知非常强大，可以决定目标方法是否执行，什么时候执行，执行时是否需要替换方法参数，执行完毕是否需要替换返回值，可通过<aop:aspect>标签下的<aop:around >标签声明：

 ```xml
<aop:around pointcut="切入点表达式"  pointcut-ref="切入点Bean引用"  
                     method="后置最终通知实现方法名"  
                     arg-names="后置最终通知实现方法参数列表参数名字"/>  
 ```

​         **pointcut和pointcut-ref****：**同前置通知同义；

​         **method：**同前置通知同义；

​         **arg-names：**同前置通知同义；

 环绕通知第一个参数必须是org.aspectj.lang.ProceedingJoinPoint类型，在通知实现方法内部使用ProceedingJoinPoint的proceed()方法使目标方法执行，proceed 方法可以传入可选的Object[]数组，该数组的值将被作为目标方法执行时的参数。

 首先在cn.javass.spring.chapter6.service.IhelloWorldService定义一个测试方法：

 ```java
public void sayAround(String param);  
 ```

其次在cn.javass.spring.chapter6.service.impl. HelloWorldService定义实现

 ```java
@Override  
public void sayAround(String param) {  
   System.out.println("============around param:" + param);  
}  
 ```

第三在cn.javass.spring.chapter6.aop. HelloWorldAspect定义通知实现：

 ```java
public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {  
    System.out.println("===========around before advice");  
    Object retVal = pjp.proceed(new Object[] {"replace"});  
    System.out.println("===========around after advice");  
    return retVal;  
}  
 ```

最后在chapter6/advice.xml配置文件中接着前置通知配置的例子添加如下配置：

 ```xml
            <aop:around pointcut="execution(* com.zero..*.sayAround(..))"
                        method="aroundAdvice"/>
 ```

测试代码cn.javass.spring.chapter6.AopTest:

 ```java
@Test  
public void testSchemaAroundAdvice() {  
    System.out.println("======================================");  
    ApplicationContext ctx = new ClassPathXmlApplicationContext("chapter6/advice.xml");  
    IHelloWorldService helloworldService =  
    ctx.getBean("helloWorldService", IHelloWorldService.class);  
    helloworldService.sayAround("haha");  
    System.out.println("======================================");  
}  
 ```

 ===========================

======================around before advice

============around param:replace

===========around after advice

====================================== 

 分析一下吧：

**1）切入点匹配：**在配置中使用“execution(* cn.javass..*.sayAround(..))”匹配目标方法sayAround；

**2）目标方法定义：**使用method="aroundAdvice"指定环绕通知实现方法，在该实现中，第一个方法参数为pjp，类型为ProceedingJoinPoint，其中“Object retVal = pjp.proceed(new Object[] {"replace"});”，用于执行目标方法，且目标方法参数被“new Object[] {"replace"}”替换，最后返回“retVal ”返回值。

**3）测试：**我们使用“helloworldService.sayAround("haha");”传入参数为“haha”，但最终输出为“replace”，说明参数被替换了

 ```java
    public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("===========around before advice");
        Object retVal = pjp.proceed(new Object[] {"replace"});
        System.out.println("===========around after advice");
        return retVal;
    }
 ```

#### 引入

​     Spring引入允许为目标对象引入新的接口，通过在< aop:aspect>标签内使用< aop:declare-parents>标签进行引入，定义方式如下：

```xml
<aop:declare-parents  
          types-matching="AspectJ语法类型表达式"  
          implement-interface=引入的接口"               
          default-impl="引入接口的默认实现"  
          delegate-ref="引入接口的默认实现Bean引用"/>  
```

​         **types-matching：**匹配需要引入接口的目标对象的AspectJ语法类型表达式；

​         **implement-interface：**定义需要引入的接口；

​         **default-impl和delegate-ref：**定义引入接口的默认实现，二者选一，default-impl是接口的默认实现类全限定名，而delegate-ref是默认的实现的委托Bean名；

接下来让我们练习一下吧：

​    首先定义引入的接口及默认实现：

```java
public interface IIntroductionService {
    public void induct();
}
```

```java
public class IntroductiondService implements IIntroductionService {
    public void induct() {
        System.out.println("=========introduction");
    }
}
```

其次在chapter6/advice.xml配置文件中接着前置通知配置的例子添加如下配置：

 ```xml
            <aop:declare-parents
                    types-matching="com.zero..*.IHelloWorldService+"
                    implement-interface="com.zero.iin.IIntroductionService"
                    default-impl="com.zero.iin.IntroductiondService"/>
 ```

最后测试一下吧，测试代码cn.javass.spring.chapter6.AopTest：

 ```java
@Test  
public void testSchemaIntroduction() {  
    System.out.println("======================================");  
    ApplicationContext ctx = new ClassPathXmlApplicationContext("chapter6/advice.xml");  
    IIntroductionService introductionService =  
    ctx.getBean("helloWorldService", IIntroductionService.class);  
    introductionService.induct();  
    System.out.println("======================================");  
}  
 ```

将输入：

 ======================================

 =========introduction

 ======================================  

 分析一下吧：

**1）目标对象类型匹配：**使用types-matching="cn.javass..*.IHelloWorldService+"匹配IHelloWorldService接口的子类型，如HelloWorldService实现；

**2）引入接口定义：**通过implement-interface属性表示引入的接口，如“cn.javass.spring.chapter6.service.IIntroductionService”。

**3）引入接口的实现：**通过default-impl属性指定，如“cn.javass.spring.chapter6.service.impl.IntroductiondService”，也可以使用“delegate-ref”来指定实现的Bean。

**4）获取引入接口：**如使用“ctx.getBean("helloWorldService", IIntroductionService.class);”可直接获取到引入的接口。

####  Advisor

 Advisor表示只有一个通知和一个切入点的切面，由于Spring AOP都是基于AOP联盟的拦截器模型的环绕通知的，所以引入Advisor来支持各种通知类型（如前置通知等5种），Advisor概念来自于Spring1.2对AOP的支持，在AspectJ中没有相应的概念对应。

 Advisor可以使用<aop:config>标签下的<aop:advisor>标签定义：

 ```xml
<aop:advisor pointcut="切入点表达式" pointcut-ref="切入点Bean引用"  
                     advice-ref="通知API实现引用"/>  
 ```

​         **pointcut和pointcut-ref****：**二者选一，指定切入点表达式；

​         **advice-ref：**引用通知API实现Bean，如前置通知接口为MethodBeforeAdvice；

 接下来让我们看一下示例吧：

 首先在cn.javass.spring.chapter6.service.IhelloWorldService定义一个测试方法：

 ```java
public void sayAdvisorBefore(String param);  
 ```

其次在cn.javass.spring.chapter6.service.impl. HelloWorldService定义实现

 ```java
@Override  
public void sayAdvisorBefore(String param) {  
    System.out.println("============say " + param);  
}  
 ```

第三定义前置通知API实现：

 ```java
package cn.javass.spring.chapter6.aop;  
import java.lang.reflect.Method;  
import org.springframework.aop.MethodBeforeAdvice;  
public class BeforeAdviceImpl implements MethodBeforeAdvice {  
    @Override  
    public void before(Method method, Object[] args, Object target) throws Throwable {  
        System.out.println("===========before advice");  
    }  
}  
 ```

在chapter6/advice.xml配置文件中先添加通知实现Bean定义：

 ```xml
<bean id="beforeAdvice" class="cn.javass.spring.chapter6.aop.BeforeAdviceImpl"/>  
 ```

然后在<aop:config>标签下，添加Advisor定义，添加时注意顺序：

 ```xml
<aop:advisor pointcut="execution(* cn.javass..*.sayAdvisorBefore(..))"  
                     advice-ref="beforeAdvice"/>  
 ```

测试代码cn.javass.spring.chapter6.AopTest:

 ```java
@Test  
public void testSchemaAdvisor() {  
   System.out.println("======================================");  
   ApplicationContext ctx = new ClassPathXmlApplicationContext("chapter6/advice.xml");  
   IHelloWorldService helloworldService =  
   ctx.getBean("helloWorldService", IHelloWorldService.class);  
   helloworldService.sayAdvisorBefore("haha");  
   System.out.println("======================================");  
}  
 ```

将输入：

 ======================================

 ===========before advice

 ============say haha

 ======================================  

 



### 基于@AspectJ的AOP

​       Spring除了支持Schema方式配置AOP，还支持注解方式：使用@AspectJ风格的切面声明。

####  启用对@AspectJ的支持

Spring默认不支持@AspectJ风格的切面声明，为了支持需要使用如下配置：

```xml
<aop:aspectj-autoproxy/>
```

这样Spring就能发现@AspectJ风格的切面并且将切面应用到目标对象。

#### 声明切面

​       @AspectJ风格的声明切面非常简单，使用@Aspect注解进行声明：

 ```java
@Aspect()  
Public class Aspect{  
……  
}  
 ```

​       然后将该切面在配置文件中声明为Bean后，Spring就能自动识别并进行AOP方面的配置：

 ```xml
<bean id="aspect" class="……Aspect"/>  
 ```

​       该切面就是一个POJO，可以在该切面中进行切入点及通知定义，接着往下看吧。

####  声明切入点

​       @AspectJ风格的命名切入点使用org.aspectj.lang.annotation包下的@Pointcut+方法（方法必须是返回void类型）实现。

 ```java
@Pointcut(value="切入点表达式", argNames = "参数名列表")  
public void pointcutName(……) {}  
 ```

​	**value：**指定切入点表达式；

​       **argNames：**指定命名切入点方法参数列表参数名字，可以有多个用“，”分隔，这些参数将传递给通知方法同名的参数，同时比如切入点表达式“args(param)”将匹配参数类型为命名切入点方法同名参数指定的参数类型。

​       **pointcutName：**切入点名字，可以使用该名字进行引用该切入点表达式。

```java
@Pointcut(value="execution(* cn.javass..*.sayAdvisorBefore(..)) && args(param)", argNames = "param")  
public void beforePointcut(String param) {}  
```

定义了一个切入点，名字为“beforePointcut”，该切入点将匹配目标方法的第一个参数类型为通知方法实现中参数名为“param”的参数类型。

####  声明通知

​       @AspectJ风格的声明通知也支持5种通知类型：

 **一、前置通知：**使用org.aspectj.lang.annotation 包下的@Before注解声明；

 ```java
@Before(value = "切入点表达式或命名切入点", argNames = "参数列表参数名")  
 ```

​	**value：**指定切入点表达式或命名切入点；

​       **argNames：**与Schema方式配置中的同义。

接下来示例一下吧：

1、定义接口和实现，在此我们就使用Schema风格时的定义；

2、定义切面：

```java
package cn.javass.spring.chapter6.aop;  
import org.aspectj.lang.annotation.Aspect;  
@Aspect  
public class HelloWorldAspect2 {  
   
}  
```

3、定义切入点：

 ```java
@Pointcut(value="execution(* cn.javass..*.sayAdvisorBefore(..)) && args(param)", argNames = "param")  
public void beforePointcut(String param) {} 
 ```

4、定义通知：

 ```java
@Before(value = "beforePointcut(param)", argNames = "param")  
public void beforeAdvice(String param) {  
    System.out.println("===========before advice param:" + param);  
}  
 ```

5、在chapter6/advice2.xml配置文件中进行如下配置：

 ```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans  xmlns="http://www.springframework.org/schema/beans"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xmlns:aop="http://www.springframework.org/schema/aop"  
        xsi:schemaLocation="  
           http://www.springframework.org/schema/beans  
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
           http://www.springframework.org/schema/aop  
           http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">  
            
  <aop:aspectj-autoproxy/>  
  <bean id="helloWorldService"  
            class="cn.javass.spring.chapter6.service.impl.HelloWorldService"/>  
   
  <bean id="aspect"  
             class="cn.javass.spring.chapter6.aop.HelloWorldAspect2"/>  
   
</beans>  
 ```

6、测试代码cn.javass.spring.chapter6.AopTest:

 ```java
@Test  
public void testAnnotationBeforeAdvice() {  
    System.out.println("======================================");  
    ApplicationContext ctx = new ClassPathXmlApplicationContext("chapter6/advice2.xml");  
    IHelloWorldService helloworldService = ctx.getBean("helloWorldService", IHelloWorldService.class);  
    helloworldService.sayBefore("before");  
    System.out.println("======================================");  
}  
 ```

将输出：

 ==========================================

 ===========before advice param:before

 ============say before

 ==========================================  

切面、切入点、通知全部使用注解完成：

​       1）使用@Aspect将POJO声明为切面；

​       2）使用@Pointcut进行命名切入点声明，同时指定目标方法第一个参数类型必须是java.lang.String，对于其他匹配的方法但参数类型不一致的将也是不匹配的，通过argNames = "param"指定了将把该匹配的目标方法参数传递给通知同名的参数上；

​       3）使用@Before进行前置通知声明，其中value用于定义切入点表达式或引用命名切入点；

​       4）配置文件需要使用<aop:aspectj-autoproxy/>来开启注解风格的@AspectJ支持；

​       5）需要将切面注册为Bean，如“aspect”Bean；

​       6）测试代码完全一样。





**二、后置返回通知：**使用org.aspectj.lang.annotation 包下的@AfterReturning注解声明；

 ```java
@AfterReturning(  
value="切入点表达式或命名切入点",  
pointcut="切入点表达式或命名切入点",  
argNames="参数列表参数名",  
returning="返回值对应参数名")  
 ```

​       **value：**指定切入点表达式或命名切入点；

​       **pointcut：**同样是指定切入点表达式或命名切入点，如果指定了将覆盖value属性指定的，pointcut具有高优先级；

​       **argName：**与Schema方式配置中的同义；

​       **returning：**与Schema方式配置中的同义。

```java
@AfterReturning(  
    value="execution(* cn.javass..*.sayBefore(..))",  
    pointcut="execution(* cn.javass..*.sayAfterReturning(..))",  
    argNames="retVal", returning="retVal")  
public void afterReturningAdvice(Object retVal) {  
    System.out.println("===========after returning advice retVal:" + retVal);  
}  
```

其中测试代码与Schema方式几乎一样，在此就不演示了，如果需要请参考AopTest.java中的testAnnotationAfterReturningAdvice测试方法。

 **三、后置异常通知：**使用org.aspectj.lang.annotation 包下的@AfterThrowing注解声明；

 ```java
@AfterThrowing (  
value="切入点表达式或命名切入点",  
pointcut="切入点表达式或命名切入点",  
argNames="参数列表参数名",  
throwing="异常对应参数名") 
 ```

​       **value：**指定切入点表达式或命名切入点；

​       **pointcut：**同样是指定切入点表达式或命名切入点，如果指定了将覆盖value属性指定的，pointcut具有高优先级；

​       **argNames：**与Schema方式配置中的同义；

​       **throwing：**与Schema方式配置中的同义。

```java
@AfterThrowing(  
    value="execution(* cn.javass..*.sayAfterThrowing(..))",  
    argNames="exception", throwing="exception")  
public void afterThrowingAdvice(Exception exception) {  
    System.out.println("===========after throwing advice exception:" + exception);  
}  
```

其中测试代码与Schema方式几乎一样，在此就不演示了，如果需要请参考AopTest.java中的testAnnotationAfterThrowingAdvice测试方法。

 **四、后置最终通知：**使用org.aspectj.lang.annotation 包下的@After注解声明；

 ```java
@After (  
value="切入点表达式或命名切入点",  
argNames="参数列表参数名")  
 ```

​       **value：**指定切入点表达式或命名切入点；

​       **argNames：**与Schema方式配置中的同义；

```java
@After(value="execution(* cn.javass..*.sayAfterFinally(..))")  
public void afterFinallyAdvice() {  
    System.out.println("===========after finally advice");  
}  
```

其中测试代码与Schema方式几乎一样，在此就不演示了，如果需要请参考AopTest.java中的testAnnotationAfterFinallyAdvice测试方法。 

**五、环绕通知：**使用org.aspectj.lang.annotation 包下的@Around注解声明；

 ```java
@Around (  
value="切入点表达式或命名切入点",  
argNames="参数列表参数名")  
 ```

​       **value：**指定切入点表达式或命名切入点；

​       **argNames：**与Schema方式配置中的同义；

```java
@Around(value="execution(* cn.javass..*.sayAround(..))")  
public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {  
    System.out.println("===========around before advice");  
    Object retVal = pjp.proceed(new Object[] {"replace"});  
    System.out.println("===========around after advice");  
    return retVal;  
}  
```

其中测试代码与Schema方式几乎一样，在此就不演示了，如果需要请参考AopTest.java中的annotationAroundAdviceTest测试方法。 

#### 引入

​       @AspectJ风格的引入声明在切面中使用org.aspectj.lang.annotation包下的@DeclareParents声明：

 ```java
@DeclareParents(  
value=" AspectJ语法类型表达式",  
defaultImpl=引入接口的默认实现类)  
private Interface interface; 
 ```

 **value：**匹配需要引入接口的目标对象的AspectJ语法类型表达式；与Schema方式中的types-matching属性同义；

​       private Interface interface**：**指定需要引入的接口；

​       defaultImpl**：**指定引入接口的默认实现类，没有与Schema方式中的delegate-ref属性同义的定义方式；



```java
@DeclareParents(  
    value="cn.javass..*.IHelloWorldService+", defaultImpl=cn.javass.spring.chapter6.service.impl.IntroductiondService.class)  
private IIntroductionService introductionService;  
   
```

其中测试代码与Schema方式几乎一样，在此就不演示了，如果需要请参考AopTest.java中的testAnnotationIntroduction测试方法。 



