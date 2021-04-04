---
title: Maven
date: 2018-09-10 18:27:21
tags: Maven
categories: 项目管理
---

## Maven

### Maven简介

​	Apache Maven是一个软件项目管理工具。基于项目对象模型(Project Object Model,POM)的概念，Maven可用来管理项目的依赖、编译、文档等信息。

​	使用Maven管理项目时，项目依赖的jar包将不再包含在项目内，而是集中放置在用户目录下的.m2文件夹下。

### Maven的pom.xml

​	Maven是基于项目对象模型的概念运作的，所以Maven的项目都有一个pom.xml用来管理项目的依赖以及项目的编译等功能。

#### 	1、dependencies元素

`<dependencies></dependencies>`，此元素包含了多个项目依赖需要使用的`<dependency></dpendency>`

#### 	2、dependency元素

`<dependency></dpendency>`	内部通过groupId、artifactId、以及version确定唯一的依赖，有人称这三个为坐标，代码如下。

​	groupId：组织的唯一标识	(可以理解为域，cn/中国。 com/商业	com.zero)

​	artifactId： 项目的唯一标识	(中国的河南省	brh-client)

​	version： 项目的版本

groupId和artifactid就可以组成项目的唯一标示，把项目放到maven本地仓库去，就需要这两个id来查找。

#### 	3、变量定义

​	`<properties></properties>`	可定义变量在dependency中引用，代码如下。

```xml
<properties>
   <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
   <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
   <java.version>1.8</java.version>
</properties>
```

#### 	4、编译插件

​	Maven提供了编译插件，可在编译插件中涉及Java的编译级别，代码如下：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>1.7</source>
                <target>1.7</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### 	5、Maven运作方式

​	Maven会自动根据dependency中的依赖配置，直接通过互联网在Maven中心库下载相关依赖包到.m2目录下，.m2目录下是你本地Maven库。

### Maven配置

#### Maven仓库

##### 配置本地仓库地址

```xml
  <localRepository>/Users/mac/Apache/repository_wh</localRepository>
```

##### 配置阿里云镜像仓库

```xml
     	 <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
```

##### 仓库优先级

最先本地仓库--->到配置文件中指定的仓库中-->配置的镜像仓库(Ali)--->默认中央仓库(Apache)

#### JDK配置

当idea中有多个JDK的时候，就需要指定编译和运行的jdk：

### Maven工程类型

#### POM工程

POM工程是逻辑工程。用在父级聚合工程中。用来做jar包版本控制。子工程可以继承父工程

#### JAR工程

将会打包成jar，用作jar包使用。即常见的本地工程-->Java Project

#### WAR工程

将会打包成war包，发布在服务器上的工程。

### IDEA-Maven项目说明

方便定位，像依赖的jar包一样

GroupID：类似包名-》防止重名->一般是域名		com.xxx

AtifactId：项目名字

Version：版本	1.0-snapshot(快照版)

### Maven目录结构

src/main/java：java源代码

src/main/resource：主要是配置文件和资源文件

src/main/test：测试用的类

src：包含了项目所有的源代码和资源文件，以及其他项目相关的文件

target：编译后内容放置的文件夹(.class文件)

### Maven工程关系

#### 依赖关系

Maven工具基于POM(Project Object Model，项目对象模型)模式实现的。在Maven中每个项目都相当于是一个对象，对象(项目)和对象(项目)之间是有关系的。包含了：依赖，继承，聚合，实现Maven项目可以更加方便的实现导jar包，拆分项目等效果。

A工程开发或运行过程中需要B工程提供支持，即代表A工程依赖B工程。

通俗讲就是导jar包。

**好处**：

- 省去了手动添加jar包的操作
- 解决了jar包冲突，例如有两个版本的jar包会自动选其一

##### 依赖的传递性

项目A依赖项目B，项目B依赖项目C，那么项目A也依赖项目C。

例如：项目2依赖项目1，项目1依赖mybatis工程，那么项目2就可以直接使用mybatis工程。

##### 依赖的两个原则

1. 最短路径优先原则

A->B-C->D(2.0)     A->E->D(1.0)，那么D(1.0)会被使用，因为路径更短

2. 最先声明优先原则

路径长度一样时，pom文件中顺序靠前的依赖优胜。

##### 依赖范围

依赖范围就决定了你依赖的坐标在什么情况下有效，什么情况下无效：

###### compile

这是默认范围。表示依赖在编译和运行时都生效。

```xml
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
            <scope>compile</scope>
        </dependency>
```

###### provided

已提供依赖范围，使用此依赖范围的Maven依赖。典型的例子是servlet-api，编译和测试项目的时候需要该依赖，但在运行项目的时候，由于容器已经提供，就不需要Maven重复地引入一遍(如：servlet-api)

打成war包的时候就不会引入到项目里，只有编译和测试的时候有

###### runtime

运行时生效

runtime范围表明编译时不需要生效，而只有运行时生效。典型的例子是JDBC驱动实现，项目主代码的编译只需要JDK提供的JDBC接口，只有在执行测试或者运行项目的时候才需要实现上述接口的具体JDBC驱动。

###### system

使用本地某个位置的jar包而不是仓库的

系统范围与provided类似，不过你必须显示指定一个本地系统路径的JAR，此类依赖应该一直有效，Maven也不会去仓库中寻找它。但是，使用system范围依赖时必须通过systemPath元素显式地指定依赖文件的路径。

```xml
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
            <scope>system</scope>
            <systemPath></systemPath>
        </dependency>
```

###### test

test范围表明使用此依赖范围的依赖，只在编译测试代码和运行测试的时候需要，应用的正常运行不需要此类依赖。典型的例子就是JUnit，它只有在编译测试代码及运行测试的时候才需要。JUnit的jar包就在测试阶段用就行了，你导出项目的时候没有必要把junit的东西导出去。所以在junit坐标下加入scope-test

###### import

import范围只适用于pom文件中的<dependencyManagement>部分。表明指定的POM必须使用`<dependencyManagement>`部分的依赖。import只能使用在`<dependencyManagement>`的scope中。

父工程pom文件中

```xml
<properties> <mybatis-version>3.4.6</mybatis-version></properties>

<!--定义可选依赖包版本-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis-version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

子工程

先将父工程打成jar包，然后定义父工程

```xml
    <parent>
        <groupId>org.example</groupId>
        <artifactId>mavenDemo1</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../mavenDemo1/pom.xml</relativePath>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
        </dependency>
    </dependencies>
```

这时子工程就不需要再指定mybatis的版本号了，直接使用的是父工程的3.4.6；此时如果子工程如果设置了其他的版本号会覆盖父工程的。

如果在父工程设置为

```xml
    <properties>
        <mybatis-version>3.4.6</mybatis-version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis-version}</version>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

那么相当于强制的制定了版本号子工程就不能使用其他的版本号了

#### 继承关系

上面的就是继承

#### 聚合

当我们开发的工程拥有2个以上模块的时候，每个模块都是一个独立的功能集合。开发的时候每个项目都可以独立编译、测试、运行。这个时候为们就需要一个聚合工程。

在创建聚合工程的过程中，总的工程必须是一个POM工程(Maven Project)(聚合项目必须是一个pom类型的项目，jar项目war项目是没有办法做聚合工程的)，各子模块可以是任意类型模块(Maven Module)。

前提：继承

聚合包含了继承的特性。

聚合时多个项目的本质还是一个项目。这些项目被一个大的父项目包含。且这时父项目类型为pom类型。同时在父项目的pom.xml中出现<moudules>表示包含的所有子模块。

先创建一个父项目,必须是pom

```xml
    <groupId>org.example</groupId>
    <artifactId>mavenDemo1</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--    定义为pom项目-->
    <packaging>pom</packaging>

    <!--    定义版本号-->
    <properties>
        <mybatis-version>3.4.6</mybatis-version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis-version}</version>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

聚合项目可以直接在父项目右键创建moudle，子模块。

```xml
    <parent>
        <artifactId>mavenDemo1</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>child</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
        </dependency>
    </dependencies>
```

### 编译器插件

```xml
 <!--    配置maven编译插件-->
    <build>
        <plugins>
    <!--     JDK编译插件       -->
            <plugin>
    <!--    插件坐标    -->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>

                <configuration>
                <!-- 源代码使用JDK版本-->
                    <source>1.8</source>
                <!--        源代码编译为class文件的版本，需要保持跟上面版本一致-->
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### 资源拷贝插件

Maven在打包时默认只将src/main/resources里的配置文件拷贝到项目中并做打包处理，而非resource目录下的配置文件在打包时不会添加到项目中。

我们的配置文件，一般放在src/main/resources，只有这个目录下的才会被打包放在target的classes目录下；

当项目非src/main/resources下的文件也打包到classes目录下需要配置。

```xml
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <excludes>
                    <exclude>bootstrap-*.yml</exclude>
                </excludes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>bootstrap-${profile_suffix}.yml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
```

设置位置下的配置文件都会被打包了。

### Tomcat插件

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                <!-- 配置Tomcat监听端口-->
                    <port>8080</port>
                <!-- 配置项目的访问路径(Application Context)-->
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

运行使用 tomcat7:run

### 常用命令

#### install

本地安装，包含编译、打包、安装到本地仓库

编译-javac

打包-jar，将java代码打包为jar文件

安装到本地仓库-将打包到jar文件，保存到本地仓库目录中。

#### clean

清除已编译信息

删除工程中target目录

#### compile

只编译。javac命令

#### package

打包。包含编译，打包两个功能

install和package命令的区别

package:package命令完成了项目编译、单元测试、打包功能，但是没有将打包好的执行jar包(war包或其他形式的包)部署到本地maven仓库和远程maven私服仓库。

install:命令完成了项目编译、单元测试、打包功能，同时将打包好的执行jar包(war包或其他形式的包)部署到本地maven仓库但没有布署到远程maven私服仓库。

## Maven问题汇总

### 依赖问题（jar包冲突）

报错信息如下：由于两个日志文件jar包冲突导致项目无法启动，卡住

![](https://user-images.githubusercontent.com/24977343/54276414-4252a900-45c8-11e9-8f87-4b389419bbcc.png)

解决方法：根据提示来查找是那个依赖引出的这个jar包。

slf4j-log4j12-1.6.1.jar

通过依次注释掉pom.xml文件中的依赖观察得出这个jar包来自zk

```xml
 				<dependency>
           <groupId>com.101tec</groupId>
           <artifactId>zkclient</artifactId>
           <version>${zkclient.version}</version>
					<exclusions>
               <exclusion>
                   <groupId>org.slf4j</groupId>
                   <artifactId>slf4j-log4j12</artifactId>
               </exclusion>
           </exclusions>
       </dependency>
```

使用

<exclusions>排除这个依赖的传递。

在mavenB项目中引入mavenA项目依赖，通过依赖传递，会将mavenA中的jar包传递进来

如果B中不需要A中的某个jar包就可以使用此标签，这里是不使用slf4j的依赖

