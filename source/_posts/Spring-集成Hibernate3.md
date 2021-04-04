---
title: Spring--集成Hibernate3
date: 2018-05-28 17:26:07
tags: Spring
categories: 框架
---

![](https://github.com/whyathere/tuchuang/blob/master/%E7%BE%8E%E5%9B%BE/%E9%95%BF%E9%A2%88%E9%B9%BF.jpg?raw=true)

### 集成Hibernate3

​       Hibernate是全自动的ORM框架，能自动为对象生成相应SQL并透明的持久化对象到数据库。

 **Spring2.5+版本支持Hibernate 3.1+版本，不支持低版本，Spring3.0.5版本提供对Hibernate 3.6.0 Final版本支持。**

####  如何集成

Spring通过使用如下Bean进行集成Hibernate：

- LocalSessionFactoryBean ：用于支持XML映射定义读取：
- configLocation和configLocations：用于定义Hibernate配置文件位置，一般使用classpath:hibernate.cfg.xml形式指定；
- mappingLocations：用于指定Hibernate映射文件位置，如chapter8/hbm/user.hbm.xml;
- hibernateProperties：用于定义Hibernate属性，即Hibernate配置文件中的属性；
- dataSource：定义数据源；
- hibernateProperties、dataSource用于消除Hibernate配置文件，因此如果使用configLocations指定配置文件，就不要设置这两个属性了，否则会产生重复配置。推荐使用dataSource来指定数据源，而使用hibernateProperties指定Hibernate属性。
- AnnotationSessionFactoryBean：用于支持注解风格映射定义读取，该类继承LocalSessionFactoryBean并额外提供自动查找注解风格配置模型的能力；
- AnnotatedClasses：设置注解了模型类，通过注解指定映射元数据。
- packagesToScan：通过扫码指定的包获取注解模型类，而不是手工指定，如"com.zero.**.model"将扫码com.zero包及子包下的model包下所有注解模型类。

接下来学习一下Spring如何继承Hibernate吧；

**1、准备jar包：**

首先准备Spring对ORM框架支持的jar包：

org.springframework.orm-3.0.5.RELEASE.jar      //提供对ORM框架集成

下载hibernate-distribution-3.6.0.Final包，获取如下Hibernate需要的jar包：

hibernate3.jar		//核心包

lib\required\antlr-2.7.6.jar		//HQL解析时使用的包

lib\required\javassist-3.9.0.GA.jar		//字节码类库，类似与cglib

lib\required\commons-collections-3.1.jar  //对集合类型支持包，前边测试时已经提供过了，无需再拷贝该包了

lib\required\dom4j-1.6.1.jar            //xml解析包，用于解析配置使用

lib\required\jta-1.1.jar                 //JTA事务支持包

lib\jpa\hibernate-jpa-2.0-api-1.0.0.Final.jar //用于支持JPA



 下载slf4j-1.6.1.zip（http://www.slf4j.org/download.html），slf4j是日志系统门面（Simple Logging Facade for Java），用于对各种日志框架提供给一致的日志访问接口，从而能随时替换日志框架（如log4j、java.util.logging）：

 

 

将这些jar包添加到类路径中。