---
title: Shiro简介
tags: Shiro
categories: 框架
---
### Shiro简介
1. Apache Shiro 是 Java 的一个安全框架。可以帮助我们完成：认证、授权、加密、会话管理、与Web集成、缓存等。

基本功能点如下图：

![](http://dl2.iteye.com/upload/attachment/0093/9788/d59f6d02-1f45-3285-8983-4ea5f18111d5.png)

Authentication：身份认证/登录，验证用户是不是拥有相应的身份;

Authorization：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

Session Manager：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境的，也可以是如Web环境的；

Cryptography：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

Web Support：Web支持，可以非常容易的集成到Web环境；

Caching：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；

Concurrency：shiro支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；

Testing：提供测试支持；

Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

Remember Me：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

记住一点，Shiro不会去维护用户、维护权限；这些需要我们自己去设计/提供；然后通过相应的接口注入给Shiro即可。

对于一个好框架，从内部看来应该具有非常简单易于使用的API，且API契约明确；从内部来看的话，其应该有一个可扩展的框架，即非常容易插入用户自定义实现，因为任何框架都不能满足所有需求。

首先，我们从外部来看Shiro吧，即从应用程序角度的来观察如何使用Shiro完成工作。如下图：

![](http://dl2.iteye.com/upload/attachment/0093/9788/d59f6d02-1f45-3285-8983-4ea5f18111d5.png)

可以看到：应用代码直接交互的对象是Subject，也就是说Shiro的对外API核心就是Subject；其每个API的含义：

Subject：主体，代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；即一个抽象概念；所有Subject都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager；可以把Subject认为是一个门面；SecurityManager才是实际的执行者；

SecurityManager：安全管理器；即所有与安全有关的操作都会与SecurityManager交互；且它管理着所有Subject；可以看出它是Shiro的核心，它负责与后边介绍的其他组件进行交互，如果学习过SpringMVC，你可以把它看成DispatcherServlet前端控制器；

Realm：域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。

从这个意义上来说，Realm本质上是一个安全特定的DAO：它封装数据源的连接细节，并根据需要将相关数据提供给Shiro。配置Shiro时，您必须至少指定一个Realm用于认证和/或授权。SecurityManager可能配置了多个领域，但至少需要一个。

Shiro提供开箱即用的Realm连接到许多安全数据源（又名目录），如LDAP，关系数据库（JDBC），文本配置源（如INI和属性文件等）。

也就是说对于我们而言，最简单的一个Shiro应用：

1、应用代码通过Subject来进行认证和授权，而Subject又委托给SecurityManager；

2、我们需要给Shiro的SecurityManager注入Realm，从而让SecurityManager能得到合法的用户及其权限进行判断。

**从以上也可以看出，Shiro不提供维护用户/权限，而是通过Realm让开发人员自己注入。**

 接下来我们来从Shiro内部来看下Shiro的架构，如下图所示：

![](http://dl2.iteye.com/upload/attachment/0093/9792/9b959a65-799d-396e-b5f5-b4fcfe88f53c.png)

Subject:主体，可以看到主体可以是任何可以与应用交互的"用户"；记录了当前操作的用户信息，我们对这个Subject进行认证和授权。

SecurityManager：相当于SpringMVC中的DispatcherServlet或者Struts2中的FilterDispatcher；是Shiro的心脏；所有具体的交互都通过SecurityManager进行控制；它管理着所有Subject、且负责进行认证和授权、及会话、缓存的管理。

Authenticator：认证器，负责主体认证的，这是一个扩展点，如果用户觉得Shiro默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了。用户用户登录认证的，主题的认证是否通过都是有authenticator来做的。

Authrizer：授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；

Realm：可以有1个或多个Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是JDBC实现，也可以是LDAP实现，或者内存实现等等；由用户提供；注意：Shiro不知道你的用户/权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的Realm；

SessionManager：如果写过Servlet就应该知道Session的概念，Session呢需要有人去管理它的生命周期，这个组件就是SessionManager；而Shiro并不仅仅可以用在Web环境，也可以用在如普通的JavaSE环境、EJB等环境；所有呢，Shiro就抽象了一个自己的Session来管理主体与应用之间交互的数据；这样的话，比如我们在Web环境用，刚开始是一台Web服务器；接着又上了台EJB服务器；这时想把两台服务器的会话数据放到一个地方，这个时候就可以实现自己的分布式会话（如把数据放到Memcached服务器）；

SessionDAO：DAO大家都用过，数据访问对象，用于会话的CRUD，比如我们想把Session保存到数据库，那么可以实现自己的SessionDAO，通过如JDBC写到数据库；比如想把Session放到Memcached中，可以实现自己的Memcached SessionDAO；另外SessionDAO中可以使用Cache进行缓存，以提高性能；通一管理session，不论分布式还是单应用都帮你做好了，比如对session的塞入一个用户和删除一个用户，而这个是用的redis这样的缓存的话就用sessionDAO。

CacheManager：缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少去改变，放到缓存中后可以提高访问的性能。

Cryptography：密码模块，Shiro提高了一些常见的加密组件用于如密码加密/解密的。

### 身份验证

#### 配置文件模拟Shiro简单认证流程

```java
public class ShiroLoginTest {

    public void testSubjectLoginAndLogout() {
        //使用工厂模式创建一个工厂类，并且加载ini配置文件，配置文件中包含了用户主体的登录信息
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro-config/userInfo.ini");
        //创建安全管理器
        SecurityManager securityManager = factory.getInstance();
        //设置当前的环境，使用securityManager
        SecurityUtils.setSecurityManager(securityManager);
        //从当前环境中获取主体
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken("jack","123456");

        subject.login(token);
        //判斷用戶是否登录
        boolean isLogin = subject.isAuthenticated();
        System.out.println("用户的登录状态是："+isLogin);
        subject.logout();
        isLogin = subject.isAuthenticated();
        System.out.println();
        System.out.println("用户的登录状态是："+isLogin);
    }

    public static void main(String[] args) {
        new ShiroLoginTest().testSubjectLoginAndLogout();
    }
}
```

这个例子是通过工厂类来加载ini文件，然后通过subject进行匹配来验证是否有权限的。

1. 首先通过new IniSecurityManagerFactory 并指定一个ini配置文件创建一个SecurityManger工厂；
2. 接着获取SecurityMnager并绑定到SecurityUtils，这是一个全局的设置，设置一次即可。
3. 通过SecurityUtils得到Subject，其会自动绑定到当前线程；如果在web环境在请求结束时需要接触绑定；然后获取身份验证的Token，如用户名/密码；
4. 调用subject.login方法进行登录，其会自动委托给SecurityManager.login方法进行登录。
5. 如果身份验证失败请捕获AuthenticationException或其子类，常见的如： DisabledAccountException（禁用的帐号）、LockedAccountException（锁定的帐号）、UnknownAccountException（错误的帐号）、ExcessiveAttemptsException（登录失败次数过多）、IncorrectCredentialsException （错误的凭证）、ExpiredCredentialsException（过期的凭证）等，具体请查看其继承关系；对于页面的错误消息展示，最好使用如“用户名/密码错误”而不是“用户名错误”/“密码错误”，防止一些恶意用户非法扫描帐号库； 
6. 同样的调用subject.logout退出，也会自动委托给SecurityManager.logout方法退出。

**从上面的代码可以大概总结出身份验证的步骤：**

1. 收集用户身份/凭证，即用户名/密码
2. 调用Subject.login进行登记，如果失败将得到相应的AuthenticationException异常，根据异常提示用户错误信息；否则登录成功；
3. 最后调用Subject.logout进行退出操作；

如上测试的几个问题：

1. 用户名/密码是硬编码在ini配置文件，以后需要改为如数据库存储，且密码需要加密存储；
2. 用户身份Token可能不仅仅是用户名/密码，也可能还有其他的，如登录时允许用户名/邮箱/手机号同时登录。

#### 身份认证流程

![](http://dl2.iteye.com/upload/attachment/0094/0173/8d639160-cd3e-3b9c-8dd6-c7f9221827a5.png)

流程如下：

1. 首先调用Subject.login(token)进行登录，其会自动委托给Security Manager，调用之前必须通过SecurityUtils. setSecurityManager()设置；
2. SecurityManager负责真正的身份验证逻辑；它会委托给Authenticator进行身份验证；
3. Authenticator才是真正的身份验证者，Shiro API中核心的身份认证入口点，此处可以自定义插入自己的实现；
4. Authenticator可能会委托给相应的AuthenticationStrategy进行多Realm身份验证，默认ModularRealmAuthenticator会调用AuthenticationStrategy进行多Realm身份验证；
5. Authenticator会把相应的token传入Realm，从Realm获取身份验证信息，如果没有返回/抛出异常表示身份验证失败了。此处可以配置多个Realm，将按照相应的顺序及策略进行访问。

#### Realm

Realm:域，Shiro从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取响应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看出DataSource，即安全数据源。如我们之前的ini配置方式将使用org.apache.shiro.realm.text.IniRealm



org.apache.shiro.realm.Realm接口如下：   



```java
String getName(); //返回一个唯一的Realm名字  

boolean supports(AuthenticationToken token); //判断此Realm是否支持此Token  

AuthenticationInfo getAuthenticationInfo(AuthenticationToken token)  

throws AuthenticationException;  //根据Token获取认证信息  
```

Shiro默认提供的Realm

![](http://dl2.iteye.com/upload/attachment/0094/0175/34062d4e-8ac5-378a-a9e2-4845f0828292.png)

以后一般继承AuthorizingRealm（授权）即可；其继承了AuthenticatingRealm（即身份验证），而且也间接继承了CachingRealm（带有缓存实现）。其中主要默认实现如下：

**org.apache.shiro.realm.text.IniRealm**：[users]部分指定用户名/密码及其角色；[roles]部分指定角色即权限信息；

**org.apache.shiro.realm.text.PropertiesRealm**：user.username=password,role1,role2指定用户名/密码及其角色；role.role1=permission1,permission2指定角色及权限信息；

**org.apache.shiro.realm.jdbc.JdbcRealm**：通过sql查询相应的信息，如“select password from users where username = ?”获取用户密码，“select password, password_salt from users where username = ?”获取用户密码及盐；“select role_name from user_roles where username = ?”获取用户角色；“select permission from roles_permissions where role_name = ?”获取角色对应的权限信息；也可以调用相应的api进行自定义sql；

#### JDBC Realm使用

##### ini配置（shiro-jdbc-realm.ini）

```ini
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql://localhost:3306/shiro
dataSource.username=root
#dataSource.password=123
jdbcRealm.dataSource=$dataSource
securityManager.realms=$jdbcRealm
```

#### Authenticator及AuthenticationStrategy

Authentication的职责是验证用户账户，是Shiro API中身份验证核心的入口点：

```java
public AuthenticationInfo authenticate(AuthenticationToken authenticationToken)  
            throws AuthenticationException;   
```

如果验证成功，将返回AuthenticationInfo验证信息；此信息中包含了身份及凭证；如果验证失败将抛出相应的AuthenticationException实现。 

SecurityManager接口继承了Authenticator，另外还有一个ModularRealmAuthenticator实现，其委托给多个Realm进行验证，验证规则通过AuthenticationStrategy接口指定，默认提供的实现：

**FirstSuccessfulStrategy**：只要有一个Realm验证成功即可，只返回第一个Realm身份验证成功的认证信息，其他的忽略；

**AtLeastOneSuccessfulStrategy**：只要有一个Realm验证成功即可，和FirstSuccessfulStrategy不同，返回所有Realm身份验证成功的认证信息；

**AllSuccessfulStrategy**：所有Realm验证成功才算成功，且返回所有Realm身份验证成功的认证信息，如果有一个失败就失败了。

ModularRealmAuthenticator默认使用AtLeastOneSuccessfulStrategy策略。

###  授权

授权，也叫访问控制，即在应用中控制谁能访问哪些资源（如访问页面/编辑数据/页面操作等）。在授权中需了解的几个关键对象：主体（Subject）、资源（Resource）、权限（Permission）、角色（Role）。 

**主体**

主体，即访问应用的用户，在Shiro中使用Subject代表该用户。用户只有授权后才允许访问相应的资源。

**资源**

在应用中用户可以访问的任何东西，比如访问JSP页面、查看/编辑某些数据、访问某个业务方法、打印文本等等都是资源。用户只要授权后才能访问。

**权限**

安全策略中的原子授权单位，通过权限我们可以表示在应用中用户有没有操作某个资源的权力。即权限表示在应用中用户能不能访问某个资源，如：

访问用户列表页面

查看/新增/修改/删除用户数据（即很多时候都是CRUD（增查改删）式权限控制）

打印文档等等。。。

如上可以看出，权限代表了用户有没有操作某个资源的权利，即反映在某个资源上的操作允不允许，不反映谁去执行这个操作。所以后续还需要把权限赋予给用户，即定义哪个用户允许在某个资源上做什么操作（权限），Shiro不会去做这件事情，而是由实现人员提供。 

Shiro支持粗粒度权限（如用户模块的所有权限）和细粒度权限（操作某个用户的权限，即实例级别的），后续部分介绍。 

**角色**

角色代表了操作集合，可以理解为权限的集合，一般情况下我们会赋予用户角色而不是权限，即这样用户可以拥有一组权限，赋予权限时比较方便。典型的如：项目经理、技术总监、CTO、开发工程师等都是角色，不同的角色拥有一组不同的权限。

**隐式角色**：即直接通过角色来验证用户有没有操作权限，如在应用中CTO、技术总监、开发工程师可以使用打印机，假设某天不允许开发工程师使用打印机，此时需要从应用中删除相应代码；再如在应用中CTO、技术总监可以查看用户、查看权限；突然有一天不允许技术总监查看用户、查看权限了，需要在相关代码中把技术总监角色从判断逻辑中删除掉；即粒度是以角色为单位进行访问控制的，粒度较粗；如果进行修改可能造成多处代码修改。 

**显示角色**：在程序中通过权限控制谁能访问某个资源，角色聚合一组权限集合；这样假设哪个角色不能访问某个资源，只需要从角色代表的权限集合中移除即可；无须修改多处代码；即粒度是以资源/实例为单位的；粒度较细。 

#### 授权方式

Shiro支持三种方式的授权：

1. Java代码方式

```java
Subject subject = SecurityUtils.getSubject();
if (subject.hasRole("admin")){
    //有权限
}else {
    //没权限
}
```

2. 注解式:通过在执行的Java方法上放置相应的注解完成：

```java
@RequiresRoles("admin")  
public void hello() {  
    //有权限  
}  
```

没有权限将抛出相应的异常；

3. JSP/GSP标签：在JSP/GSP页面通过相应的标签完成：

```javascript
<shiro:hasRole name="admin">  
<!— 有权限 —>  
</shiro:hasRole>   
```

后续部分将详细介绍如何使用。   

####  授权

**基于角色的访问控制（隐式角色）**

 ```ini
[users]  
zhang=123,role1,role2  
wang=123,role1 
 ```

规则：“用户名=密码,角色1，角色2”，如果需要在应用中判断用户是否有相应角色，就需要在相应的Realm中返回角色信息，也就是说Shiro不负责维护用户-角色信息，需要应用提供，Shiro只是提供相应的接口方便验证，后续会介绍如何动态的获取用户角色。 

测试代码

```java
@Test  
public void testHasRole() {  
    login("classpath:shiro-role.ini", "zhang", "123");  
    //判断拥有角色：role1  
    Assert.assertTrue(subject().hasRole("role1"));  
    //判断拥有角色：role1 and role2  
    Assert.assertTrue(subject().hasAllRoles(Arrays.asList("role1", "role2")));  
    //判断拥有角色：role1 and role2 and !role3  
    boolean[] result = subject().hasRoles(Arrays.asList("role1", "role2", "role3"));  
    Assert.assertEquals(true, result[0]);  
    Assert.assertEquals(true, result[1]);  
    Assert.assertEquals(false, result[2]);  
}  
```

Shiro提供了hasRole/hasRole用于判断用户是否拥有某个角色/某些权限；但是没有提供如hashAnyRole用于判断是否有某些权限中的某一个。  

```java
@Test(expected = UnauthorizedException.class)  
public void testCheckRole() {  
    login("classpath:shiro-role.ini", "zhang", "123");  
    //断言拥有角色：role1  
    subject().checkRole("role1");  
    //断言拥有角色：role1 and role3 失败抛出异常  
    subject().checkRoles("role1", "role3");  
}   
```

Shiro提供的checkRole/checkRoles和hasRole/hasAllRoles不同的地方是它在判断为假的情况下会抛出UnauthorizedException异常。 

到此基于角色的访问控制（即隐式角色）就完成了，这种方式的缺点就是如果很多地方进行了角色判断，但是有一天不需要了那么就需要修改相应代码把所有相关的地方进行删除；这就是粗粒度造成的问题。 

**基于资源的访问控制（显示角色** )

1、在ini配置文件配置用户拥有的角色及角色-权限关系（shiro-permission.ini）  

```ini
[users]  
zhang=123,role1,role2  
wang=123,role1  
[roles]  
role1=user:create,user:update  
role2=user:create,user:delete 
```

规则：“用户名=密码，角色1，角色2”“角色=权限1，权限2”，即首先根据用户名找到角色，然后根据角色再找到权限；即角色是权限集合；Shiro同样不进行权限的维护，需要我们通过Realm返回相应的权限信息。只需要维护“用户——角色”之间的关系即可。 

```java
@Test  
public void testIsPermitted() {  
    login("classpath:shiro-permission.ini", "zhang", "123");  
    //判断拥有权限：user:create  
    Assert.assertTrue(subject().isPermitted("user:create"));  
    //判断拥有权限：user:update and user:delete  
    Assert.assertTrue(subject().isPermittedAll("user:update", "user:delete"));  
    //判断没有权限：user:view  
    Assert.assertFalse(subject().isPermitted("user:view"));  
}   
```

Shiro提供了isPermitted和isPermittedAll用于判断用户是否拥有某个权限或所有权限，也没有提供如isPermittedAny用于判断拥有某一个权限的接口。 

```java
@Test(expected = UnauthorizedException.class)  
public void testCheckPermission () {  
    login("classpath:shiro-permission.ini", "zhang", "123");  
    //断言拥有权限：user:create  
    subject().checkPermission("user:create");  
    //断言拥有权限：user:delete and user:update  
    subject().checkPermissions("user:delete", "user:update");  
    //断言拥有权限：user:view 失败抛出异常  
    subject().checkPermissions("user:view");  
}  
```

但是失败的情况下会抛出UnauthorizedException异常。

 到此基于资源的访问控制（显示角色）就完成了，也可以叫基于权限的访问控制，这种方式的一般规则是“资源标识符：操作”，即是资源级别的粒度；这种方式的好处就是如果要修改基本都是一个资源级别的修改，不会对其他模块代码产生影响，粒度小。但是实现起来可能稍微复杂点，需要维护“用户——角色，角色——权限（资源：操作）”之间的关系。  

####  Permission

字符串通配符权限

 规则：“资源标识符：操作：对象实例ID”  即对哪个资源的哪个实例可以进行什么操作。其默认支持通配符权限字符串，“:”表示资源/操作/实例的分割；“,”表示操作的分割；“*”表示任意资源/操作/实例。

1. 单个资源单个权限

```java
subject().checkPermissions("system:user:update");  
```

2. 单个资源多个权限

ini配置文件：

```ini
role41=system:user:update,system:user:delete  
```

然后通过如下代码判断  

 ```java
subject().checkPermissions("system:user:update", "system:user:delete");  
 ```

用户拥有资源“system:user”的“update”和“delete”权限。如上可以简写成：

 

ini配置（表示角色4拥有system:user资源的update和delete权限）     

```ini
role42="system:user:update,delete"    
```

 接着可以通过如下代码判断   

```java
subject().checkPermissions("system:user:update,delete");  
```

通过“system:user:update,delete”验证"system:user:update, system:user:delete"是没问题的，但是反过来是规则不成立。

3. 单个资源全部权限

ini配置   

```ini
role51="system:user:create,update,delete,view"  
```

然后通过如下代码判断   

```java
subject().checkPermissions("system:user:create,delete,update:view");  
```

用户拥有资源“system:user”的“create”、“update”、“delete”和“view”所有权限。如上可以简写成：

 

ini配置文件（表示角色5拥有system:user的所有权限）  

```ini
role52=system:user:*  
```

也可以简写为（推荐上边的写法）：  

```ini
role53=system:user  
```

然后通过如下代码判断  

```java
subject().checkPermissions("system:user:*");  
subject().checkPermissions("system:user");
```

通过“system:user:*”验证“system:user:create,delete,update:view”可以，但是反过来是不成立的。

4. **所有资源全部权限** 

ini配置   

```ini
role61=*:view  
```

然后通过如下代码判断  

```java
subject().checkPermissions("user:view");  
```

用户拥有所有资源的“view”所有权限。假设判断的权限是“"system:user:view”，那么需要“role5=*:*:view”这样写才行。

5. **实例级别的权限** 

**5.1、单个实例单个权限**

 ini配置  

```ini
role71=user:view:1  
```

对资源user的1实例拥有view权限。

然后通过如下代码判断 

```java
subject().checkPermissions("user:view:1");  
```

**5.2、单个实例多个权限**

 ini配置             

```ini
role72="user:update,delete:1"  
```

对资源user的1实例拥有update、delete权限。

然后通过如下代码判断

```java
subject().checkPermissions("user:delete,update:1");  
subject().checkPermissions("user:update:1", "user:delete:1");   
```

**5.3、单个实例所有权限** 

ini配置    

```ini
role73=user:*:1  
```

对资源user的1实例拥有所有权限。

然后通过如下代码判断 

```java
subject().checkPermissions("user:update:1", "user:delete:1", "user:view:1");  
```

**5.4、所有实例单个权限** 

ini配置   

```ini
role74=user:auth:*  
```

对资源user的1实例拥有所有权限。

然后通过如下代码判断 

```java
subject().checkPermissions("user:auth:1", "user:auth:2");  
```

**5.5、所有实例所有权限**

 ini配置   

```ini
role75=user:*:*  
```

对资源user的1实例拥有所有权限。

然后通过如下代码判断 

```java
subject().checkPermissions("user:view:1", "user:auth:2");  
```

**6、Shiro对权限字符串缺失部分的处理** 

如“user:view”等价于“user:view:*”；而“organization”等价于“organization:*”或者“organization:*:*”。可以这么理解，这种方式实现了前缀匹配。

另外如“user:*”可以匹配如“user:delete”、“user:delete”可以匹配如“user:delete:1”、“user:*:1”可以匹配如“user:view:1”、“user”可以匹配“user:view”或“user:view:1”等。即*可以匹配所有，不加*可以进行前缀匹配；但是如“*:view”不能匹配“system:user:view”，需要使用“*:*:view”，即后缀匹配必须指定前缀（多个冒号就需要多个*来匹配）。

**7、WildcardPermission**

 如下两种方式是等价的：    

```java
subject().checkPermission("menu:view:1");  
subject().checkPermission(new WildcardPermission("menu:view:1"));   
```

因此没什么必要的话使用字符串更方便。

 **8、性能问题**

通配符匹配方式比字符串相等匹配来说是更复杂的，因此需要花费更长时间，但是一般系统的权限不会太多，且可以配合缓存来提供其性能，如果这样性能还达不到要求我们可以实现位操作算法实现性能更好的权限匹配。另外实例级别的权限验证如果数据量太大也不建议使用，可能造成查询权限及匹配变慢。可以考虑比如在sql查询时加上权限字符串之类的方式在查询时就完成了权限匹配。

#### 授权流程

 ![](http://dl2.iteye.com/upload/attachment/0094/0549/541e4da3-d1a5-3d13-83a6-b65c3596ee4e.png)

流程如下：

1、首先调用Subject.isPermitted*/hasRole*接口，其会委托给SecurityManager，而SecurityManager接着会委托给Authorizer；

2、Authorizer是真正的授权者，如果我们调用如isPermitted(“user:view”)，其首先会通过PermissionResolver把字符串转换成相应的Permission实例；

3、在进行授权之前，其会调用相应的Realm获取Subject相应的角色/权限用于匹配传入的角色/权限；

4、Authorizer会判断Realm的角色/权限是否和传入的匹配，如果有多个Realm，会委托给ModularRealmAuthorizer进行循环判断，如果匹配如isPermitted*/hasRole*会返回true，否则返回false表示授权失败。

ModularRealmAuthorizer进行多Realm匹配流程：

1、首先检查相应的Realm是否实现了实现了Authorizer；

2、如果实现了Authorizer，那么接着调用其相应的isPermitted*/hasRole*接口进行匹配；

3、如果有一个Realm匹配那么将返回true，否则返回false。

 

如果Realm进行授权的话，应该继承AuthorizingRealm，其流程是：

1.1、如果调用hasRole*，则直接获取AuthorizationInfo.getRoles()与传入的角色比较即可；

1.2、首先如果调用如isPermitted(“user:view”)，首先通过PermissionResolver将权限字符串转换成相应的Permission实例，默认使用WildcardPermissionResolver，即转换为通配符的WildcardPermission；

2、通过AuthorizationInfo.getObjectPermissions()得到Permission实例集合；通过AuthorizationInfo. getStringPermissions()得到字符串集合并通过PermissionResolver解析为Permission实例；然后获取用户的角色，并通过RolePermissionResolver解析角色对应的权限集合（默认没有实现，可以自己提供）；

3、接着调用Permission. implies(Permission p)逐个与传入的权限比较，如果有匹配的则返回true，否则false。 

#### Authorizer、PermissionResolver及RolePermissionResolver

Authorizer的职责是进行授权（访问控制），是Shiro API中授权核心的入口点，其提供了相应的角色/权限判断接口，具体请参考其Javadoc。SecurityManager继承了Authorizer接口，且提供了ModularRealmAuthorizer用于多Realm时的授权匹配。PermissionResolver用于解析权限字符串到Permission实例，而RolePermissionResolver用于根据角色解析相应的权限集合。 

我们可以通过如下ini配置更改Authorizer实现：   

```ini
authorizer=org.apache.shiro.authz.ModularRealmAuthorizer  
securityManager.authorizer=$authorizer  
```

对于ModularRealmAuthorizer，相应的AuthorizingSecurityManager会在初始化完成后自动将相应的realm设置进去，我们也可以通过调用其setRealms()方法进行设置。对于实现自己的authorizer可以参考ModularRealmAuthorizer实现即可，在此就不提供示例了。

 

设置ModularRealmAuthorizer的permissionResolver，其会自动设置到相应的Realm上（其实现了PermissionResolverAware接口），如： 

```ini
permissionResolver=org.apache.shiro.authz.permission.WildcardPermissionResolver  
authorizer.permissionResolver=$permissionResolver  
```

设置ModularRealmAuthorizer的rolePermissionResolver，其会自动设置到相应的Realm上（其实现了RolePermissionResolverAware接口），如： 

```ini
rolePermissionResolver=com.github.zhangkaitao.shiro.chapter3.permission.MyRolePermissionResolver  
authorizer.rolePermissionResolver=$rolePermissionResolver  
```

**示例** 

**1、ini配置（shiro-authorizer.ini）** 

```ini
[main]  
#自定义authorizer  
authorizer=org.apache.shiro.authz.ModularRealmAuthorizer  
#自定义permissionResolver  
#permissionResolver=org.apache.shiro.authz.permission.WildcardPermissionResolver  
permissionResolver=com.github.zhangkaitao.shiro.chapter3.permission.BitAndWildPermissionResolver  
authorizer.permissionResolver=$permissionResolver  
#自定义rolePermissionResolver  
rolePermissionResolver=com.github.zhangkaitao.shiro.chapter3.permission.MyRolePermissionResolver  
authorizer.rolePermissionResolver=$rolePermissionResolver  
  
securityManager.authorizer=$authorizer  
```

```ini
#自定义realm 一定要放在securityManager.authorizer赋值之后（因为调用setRealms会将realms设置给authorizer，并给各个Realm设置permissionResolver和rolePermissionResolver）  
realm=com.github.zhangkaitao.shiro.chapter3.realm.MyRealm  
securityManager.realms=$realm   
```

设置securityManager 的realms一定要放到最后，因为在调用SecurityManager.setRealms时会将realms设置给authorizer，并为各个Realm设置permissionResolver和rolePermissionResolver。另外，不能使用IniSecurityManagerFactory创建的IniRealm，因为其初始化顺序的问题可能造成后续的初始化Permission造成影响。 

##### **2、定义BitAndWildPermissionResolver及BitPermission**

BitPermission用于实现位移方式的权限，如规则是：

权限字符串格式：+资源字符串+权限位+实例ID；以+开头中间通过+分割；权限：0 表示所有权限；1 新增（二进制：0001）、2 修改（二进制：0010）、4 删除（二进制：0100）、8 查看（二进制：1000）；如 +user+10 表示对资源user拥有修改/查看权限。

```java
public class BitPermission implements Permission {  
    private String resourceIdentify;  
    private int permissionBit;  
    private String instanceId;  
    public BitPermission(String permissionString) {  
        String[] array = permissionString.split("\\+");  
        if(array.length > 1) {  
            resourceIdentify = array[1];  
        }  
        if(StringUtils.isEmpty(resourceIdentify)) {  
            resourceIdentify = "*";  
        }  
        if(array.length > 2) {  
            permissionBit = Integer.valueOf(array[2]);  
        }  
        if(array.length > 3) {  
            instanceId = array[3];  
        }  
        if(StringUtils.isEmpty(instanceId)) {  
            instanceId = "*";  
        }  
    }  
  
    @Override  
    public boolean implies(Permission p) {  
        if(!(p instanceof BitPermission)) {  
            return false;  
        }  
        BitPermission other = (BitPermission) p;  
        if(!("*".equals(this.resourceIdentify) || this.resourceIdentify.equals(other.resourceIdentify))) {  
            return false;  
        }  
        if(!(this.permissionBit ==0 || (this.permissionBit & other.permissionBit) != 0)) {  
            return false;  
        }  
        if(!("*".equals(this.instanceId) || this.instanceId.equals(other.instanceId))) {  
            return false;  
        }  
        return true;  
    }  
}  
```

Permission接口提供了boolean implies(Permission p)方法用于判断权限匹配的；  

```java
public class BitAndWildPermissionResolver implements PermissionResolver {  
    @Override  
    public Permission resolvePermission(String permissionString) {  
        if(permissionString.startsWith("+")) {  
            return new BitPermission(permissionString);  
        }  
        return new WildcardPermission(permissionString);  
    }  
}  
```

BitAndWildPermissionResolver实现了PermissionResolver接口，并根据权限字符串是否以“+”开头来解析权限字符串为BitPermission或WildcardPermission。

#####  **定义MyRolePermissionResolver** 

RolePermissionResolver用于根据角色字符串来解析得到权限集合。  

```java
public class MyRolePermissionResolver implements RolePermissionResolver {  
    @Override  
    public Collection<Permission> resolvePermissionsInRole(String roleString) {  
        if("role1".equals(roleString)) {  
            return Arrays.asList((Permission)new WildcardPermission("menu:*"));  
        }  
        return null;  
    }  
}  
```

此处的实现很简单，如果用户拥有role1，那么就返回一个“menu:*”的权限。

#####  **自定义Realm** 

```java
public class MyRealm extends AuthorizingRealm {  
    @Override  
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {  
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();  
        authorizationInfo.addRole("role1");  
        authorizationInfo.addRole("role2");  
        authorizationInfo.addObjectPermission(new BitPermission("+user1+10"));  
        authorizationInfo.addObjectPermission(new WildcardPermission("user1:*"));  
        authorizationInfo.addStringPermission("+user2+10");  
        authorizationInfo.addStringPermission("user2:*");  
        return authorizationInfo;  
    }  
    @Override  
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {  
        //和com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1. getAuthenticationInfo代码一样，省略  
}  
}   
```

此时我们继承AuthorizingRealm而不是实现Realm接口；推荐使用AuthorizingRealm，因为：

AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token)：表示获取身份验证信息；

AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals)：表示根据用户身份获取授权信息。

这种方式的好处是当只需要身份验证时只需要获取身份验证信息而不需要获取授权信息。对于AuthenticationInfo和AuthorizationInfo请参考其Javadoc获取相关接口信息。

另外我们可以使用JdbcRealm，需要做的操作如下：

1、执行sql/ shiro-init-data.sql 插入相关的权限数据；

2、使用shiro-jdbc-authorizer.ini配置文件，需要设置jdbcRealm.permissionsLookupEnabled

为true来开启权限查询。

 

此次还要注意就是不能把我们自定义的如“+user1+10”配置到INI配置文件，即使有IniRealm完成，因为IniRealm在new完成后就会解析这些权限字符串，默认使用了WildcardPermissionResolver完成，即此处是一个设计权限，如果采用生命周期（如使用初始化方法）的方式进行加载就可以解决我们自定义permissionResolver的问题。

```java
public class AuthorizerTest extends BaseTest {  
  
    @Test  
    public void testIsPermitted() {  
        login("classpath:shiro-authorizer.ini", "zhang", "123");  
        //判断拥有权限：user:create  
        Assert.assertTrue(subject().isPermitted("user1:update"));  
        Assert.assertTrue(subject().isPermitted("user2:update"));  
        //通过二进制位的方式表示权限  
        Assert.assertTrue(subject().isPermitted("+user1+2"));//新增权限  
        Assert.assertTrue(subject().isPermitted("+user1+8"));//查看权限  
        Assert.assertTrue(subject().isPermitted("+user2+10"));//新增及查看  
  
        Assert.assertFalse(subject().isPermitted("+user1+4"));//没有删除权限  
  
        Assert.assertTrue(subject().isPermitted("menu:view"));//通过MyRolePermissionResolver解析得到的权限  
    }  
}   
```

### INI配置

之前章节我们已经接触过一些INI配置规则了，如果大家使用过如Spring之类的IoC/DI容器的话，Shiro提供的INI配置也是非常类似的，即可以理解为是一个IoC/DI容器，但是区别在于它从一个根对象securityManager开始。

#### 根对象SecurityManager

 从之前的Shiro架构图可以看出，Shiro是从根对象SecurityManager进行身份验证和授权的；也就是所有操作都是自它开始的，这个对象是线程安全且整个应用只需要一个即可，因此Shiro提供了SecurityUtils让我们绑定它为全局的，方便后续操作。

 因为Shiro的类都是POJO的，因此都很容易放到任何IoC容器管理。但是和一般的IoC容器的区别在于，Shiro从根对象securityManager开始导航；Shiro支持的依赖注入：public空参构造器对象的创建、setter依赖注入。

 1、纯Java代码写法 

```java
DefaultSecurityManager securityManager = new DefaultSecurityManager();  
//设置authenticator  
ModularRealmAuthenticator authenticator = new ModularRealmAuthenticator();  
authenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());  
securityManager.setAuthenticator(authenticator);  
  
//设置authorizer  
ModularRealmAuthorizer authorizer = new ModularRealmAuthorizer();  
authorizer.setPermissionResolver(new WildcardPermissionResolver());  
securityManager.setAuthorizer(authorizer);  
  
//设置Realm  
DruidDataSource ds = new DruidDataSource();  
ds.setDriverClassName("com.mysql.jdbc.Driver");  
ds.setUrl("jdbc:mysql://localhost:3306/shiro");  
ds.setUsername("root");  
ds.setPassword("");  
  
JdbcRealm jdbcRealm = new JdbcRealm();  
jdbcRealm.setDataSource(ds);  
jdbcRealm.setPermissionsLookupEnabled(true);  
securityManager.setRealms(Arrays.asList((Realm) jdbcRealm));  
  
//将SecurityManager设置到SecurityUtils 方便全局使用  
SecurityUtils.setSecurityManager(securityManager);  
  
Subject subject = SecurityUtils.getSubject();  
UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");  
subject.login(token);  
Assert.assertTrue(subject.isAuthenticated());  
```

2.1、等价的INI配置（shiro-config.ini） 

```ini
[main]  
#authenticator  
authenticator=org.apache.shiro.authc.pam.ModularRealmAuthenticator  
authenticationStrategy=org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy  
authenticator.authenticationStrategy=$authenticationStrategy  
securityManager.authenticator=$authenticator  
  
#authorizer  
authorizer=org.apache.shiro.authz.ModularRealmAuthorizer  
permissionResolver=org.apache.shiro.authz.permission.WildcardPermissionResolver  
authorizer.permissionResolver=$permissionResolver  
securityManager.authorizer=$authorizer  
  
#realm  
dataSource=com.alibaba.druid.pool.DruidDataSource  
dataSource.driverClassName=com.mysql.jdbc.Driver  
dataSource.url=jdbc:mysql://localhost:3306/shiro  
dataSource.username=root  
#dataSource.password=  
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm  
jdbcRealm.dataSource=$dataSource  
jdbcRealm.permissionsLookupEnabled=true  
securityManager.realms=$jdbcRealm   
```

即使没接触过IoC容器的知识，如上配置也是很容易理解的：

1、对象名=全限定类名  相对于调用public无参构造器创建对象

2、对象名.属性名=值    相当于调用setter方法设置常量值

3、对象名.属性名=$对象引用    相当于调用setter方法设置对象引用



```java
Factory<org.apache.shiro.mgt.SecurityManager> factory =  
         new IniSecurityManagerFactory("classpath:shiro-config.ini");  
  
org.apache.shiro.mgt.SecurityManager securityManager = factory.getInstance();  
  
//将SecurityManager设置到SecurityUtils 方便全局使用  
SecurityUtils.setSecurityManager(securityManager);  
Subject subject = SecurityUtils.getSubject();  
UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");  
subject.login(token);  
  
Assert.assertTrue(subject.isAuthenticated()); 
```

如上代码是从Shiro INI配置中获取相应的securityManager实例： 

1. 默认情况先创建一个名字为securityManager，类型为org.apache.shiro.mgt.DefaultSecurityManager的默认的SecurityManager，如果想自定义，只需要在ini配置文件中指定“securityManager=SecurityManager实现类”即可，名字必须为securityManager，它是起始的根； 
2. IniSecurityManagerFactory是创建securityManager的工厂，其需要一个ini配置文件路径，其支持“classpath:”（类路径）、“file:”（文件系统）、“url:”（网络）三种路径格式，默认是文件系统； 
3. 接着获取SecuriyManager实例，后续步骤和之前的一样。 

从如上可以看出Shiro INI配置方式本身提供了一个简单的IoC/DI机制方便在配置文件配置，但是是从securityManager这个根对象开始导航。    

#### INI配置

ini配置文件类似于Java中的properties（kye=value），不过提供了将key/value分类的特性，key是每个部分不重复即可，而不是整个配置文件。如下是INI配置分类：

```ini
[main]  
#提供了对根对象securityManager及其依赖的配置  
securityManager=org.apache.shiro.mgt.DefaultSecurityManager  
…………  
securityManager.realms=$jdbcRealm  
  
[users]  
#提供了对用户/密码及其角色的配置，用户名=密码，角色1，角色2  
username=password,role1,role2  
  
[roles]  
#提供了角色及权限之间关系的配置，角色=权限1，权限2  
role1=permission1,permission2  
  
[urls]  
#用于web，提供了对web url拦截相关的配置，url=拦截器[参数]，拦截器  
/index.html = anon  
/admin/** = authc, roles[admin], perms["permission1"]  
```

**[main]****部分**

提供了对根对象securityManager及其依赖对象的配置。

**创建对象** 

```ini
securityManager=org.apache.shiro.mgt.DefaultSecurityManager  
```

其构造器必须是public空参构造器，通过反射创建相应的实例。

 **常量值setter注入**   

```ini
dataSource.driverClassName=com.mysql.jdbc.Driver  
jdbcRealm.permissionsLookupEnabled=true 
```

会自动调用jdbcRealm.setPermissionsLookupEnabled(true)，对于这种常量值会自动类型转换。

 **对象引用setter注入**   

```ini
authenticator=org.apache.shiro.authc.pam.ModularRealmAuthenticator  
authenticationStrategy=org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy  
authenticator.authenticationStrategy=$authenticationStrategy  
securityManager.authenticator=$authenticator   
```

会自动通过securityManager.setAuthenticator(authenticator)注入引用依赖。

 **嵌套属性setter注入**   

```ini
securityManager.authenticator.authenticationStrategy=$authenticationStrategy   
```

**byte数组setter注入**   

```ini
#base64 byte[]  
authenticator.bytes=aGVsbG8=  
#hex byte[]  
authenticator.bytes=0x68656c6c6f  
```

默认需要使用Base64进行编码，也可以使用0x十六进制。

 **Array/Set/List setter注入**   

```ini
authenticator.array=1,2,3  
authenticator.set=$jdbcRealm,$jdbcRealm  
```

多个之间通过“，”分割。

 **Map setter注入**  

```ini
authenticator.map=$jdbcRealm:$jdbcRealm,1:1,key:abc  
```

即格式是：map=key：value，key：value，可以注入常量及引用值，常量的话都看作字符串（即使有泛型也不会自动造型）。        

 **实例化/注入顺序**   

```ini
realm=Realm1  
realm=Realm12  
  
authenticator.bytes=aGVsbG8=  
authenticator.bytes=0x68656c6c6f   
```

后边的覆盖前边的注入。

 测试用例请参考配置文件shiro-config-main.ini。 

 **[users]部分**

配置用户名/密码及其角色，格式：“用户名=密码，角色1，角色2”，角色部分可省略。如：

```ini
[users]  
zhang=123,role1,role2  
wang=123
```

密码一般生成其摘要/加密存储，后续章节介绍。

 **[roles]部分**

配置角色及权限之间的关系，格式：“角色=权限1，权限2”；如：

```ini
[roles]  
role1=user:create,user:update  
role2=*   
```

如果只有角色没有对应的权限，可以不配roles，具体规则请参考授权章节。

 **[urls]部分**

 配置url及相应的拦截器之间的关系，格式：“url=拦截器[参数]，拦截器[参数]，如：    

```ini
[urls]  
/admin/** = authc, roles[admin], perms["permission1"]   
```

### 编码/加密

在涉及到密码存储问题上，应该加密/生成密码摘要存储，而不是存储明文密码。比如之前的600w csdn账号泄露对用户可能造成很大损失，因此应加密/生成不可逆的摘要方式存储。

####  编码/解码 

Shiro提供了base64和16进制字符串编码/解码的API支持，方便一些编码解码操作。Shiro内部的一些数据的存储/表示都使用了base64和16进制字符串。  

```java
String str = "hello";  
String base64Encoded = Base64.encodeToString(str.getBytes()); //编码 
String str2 = Base64.decodeToString(base64Encoded);  //解码
Assert.assertEquals(str, str2); 
```

通过如上方式可以进行16进制字符串编码/解码操作，更多API请参考其Javadoc。

还有一个可能经常用到的类CodecSupport，提供了toBytes(str, "utf-8") / toString(bytes, "utf-8")用于在byte数组/String之间转换。

####   散列算法

散列算法一般用于生成数据的摘要信息，是一种不可逆算法，一般适合存储密码之类的数据，常见的散列算法如MD5、SHA等。一般进行散列时最好提供一个salt（盐），比如加密密码"admin"，产生的散列值是"21232f297a57a5a743894a0e4a801fc3 ",可以到一些MD5解密网站很容易的通过散列值得到密码"admin"，即如果直接对密码进行散列相对来说破解更容易，此时我们可以加一些只有系统知道的干扰数据，如用户名和ID（即盐）；这样散列的对象是“密码+用户名+ID”，这样生成的散列值相对来说更难破解。

```java
String str = "hello";  
String salt = "123";  
String md5 = new Md5Hash(str, salt).toString();//还可以转换为 toBase64()/toHex()   
```

如上代码通过盐“123”MD5散列“hello”。另外散列时还可以指定散列次数，如2次表示：md5(md5(str))：“new Md5Hash(str, salt, 2).toString()” 

```java
String str = "hello";  
String salt = "123";  
String sha1 = new Sha256Hash(str, salt).toString();   
```

使用SHA256算法生成相应的散列数据，另外还有如SHA1、SHA512算法。     

 Shiro还提供了通用的散列支持：  

```java
String str = "hello";  
String salt = "123";  
//内部使用MessageDigest  
String simpleHash = new SimpleHash("SHA-1", str, salt).toString(); 
```

通过调用SimpleHash时指定散列算法，其内部使用了Java的MessageDigest实现。



 为了方便使用，Shiro提供了HashService，默认提供了DefaultHashService实现。  

```java
DefaultHashService hashService = new DefaultHashService(); //默认算法SHA-512  
hashService.setHashAlgorithmName("SHA-512");  
hashService.setPrivateSalt(new SimpleByteSource("123")); //私盐，默认无  
hashService.setGeneratePublicSalt(true);//是否生成公盐，默认false  
hashService.setRandomNumberGenerator(new SecureRandomNumberGenerator());//用于生成公盐。默认就这个  
hashService.setHashIterations(1); //生成Hash值的迭代次数  
  
HashRequest request = new HashRequest.Builder()  
            .setAlgorithmName("MD5").setSource(ByteSource.Util.bytes("hello"))  
            .setSalt(ByteSource.Util.bytes("123")).setIterations(2).build();  
String hex = hashService.computeHash(request).toHex();   
```

1. 首先创建一个DefaultHashService，默认使用SHA-512算法；
2. 可以通过hashAlgorithmName属性修改算法；
3. 可以通过privateSalt设置一个私盐，其在散列时自动与用户传入的公盐混合产生一个新盐；
4. 可以通过generatePublicSalt属性在用户没有传入公盐的情况下是否生成公盐；
5. 可以设置randomNumberGenerator用于生成公盐；
6. 可以设置hashIterations属性来修改默认加密迭代次数；
7. 需要构建一个HashRequest，传入算法、数据、公盐、迭代次数。



SecureRandomNumberGenerator用于生成一个随机数：  

```java
SecureRandomNumberGenerator randomNumberGenerator =  
     new SecureRandomNumberGenerator();  
randomNumberGenerator.setSeed("123".getBytes());  
String hex = randomNumberGenerator.nextBytes().toHex(); 
```

#### 加密/解密

Shiro还提供对称式加密/解密算法的支持，如AES、Blowfish等；当前还没有提供对非对称加密/解密算法支持，未来版本可能提供。

 AES算法实现：  

```java
AesCipherService aesCipherService = new AesCipherService();  
aesCipherService.setKeySize(128); //设置key长度  
//生成key  
Key key = aesCipherService.generateNewKey();  
String text = "hello";  
//加密  
String encrptText =   
aesCipherService.encrypt(text.getBytes(), key.getEncoded()).toHex();  
//解密  
String text2 =  
 new String(aesCipherService.decrypt(Hex.decode(encrptText), key.getEncoded()).getBytes());  
  
Assert.assertEquals(text, text2); 
```

#### PasswordService/CredentialsMatcher

Shiro提供了PasswordService及CredentialsMatcher用于提供加密密码及验证密码服务。  

```java
public interface PasswordService {  
    //输入明文密码得到密文密码  
    String encryptPassword(Object plaintextPassword) throws IllegalArgumentException;  
}  
```

```java
public interface CredentialsMatcher {  
    //匹配用户输入的token的凭证（未加密）与系统提供的凭证（已加密）  
    boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info);  
}   
```

Shiro默认提供了PasswordService实现DefaultPasswordService；CredentialsMatcher实现PasswordMatcher及HashedCredentialsMatcher（更强大）。

 **DefaultPasswordService配合PasswordMatcher实现简单的密码加密与验证服务**

1. 定义Realm（com.github.zhangkaitao.shiro.chapter5.hash.realm.MyRealm） 

```java
public class MyRealm extends AuthorizingRealm {  
    private PasswordService passwordService;  
    public void setPasswordService(PasswordService passwordService) {  
        this.passwordService = passwordService;  
    }  
     //省略doGetAuthorizationInfo，具体看代码   
    @Override  
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {  
        return new SimpleAuthenticationInfo(  
                "wu",  
                passwordService.encryptPassword("123"),  
                getName());  
    }  
}   
```

为了方便，直接注入一个passwordService来加密密码，实际使用时需要在Service层使用passwordService加密密码并存到数据库。

 2、ini配置（shiro-passwordservice.ini）  

```ini
[main]  
passwordService=org.apache.shiro.authc.credential.DefaultPasswordService  
hashService=org.apache.shiro.crypto.hash.DefaultHashService  
passwordService.hashService=$hashService  
hashFormat=org.apache.shiro.crypto.hash.format.Shiro1CryptFormat  
passwordService.hashFormat=$hashFormat  
hashFormatFactory=org.apache.shiro.crypto.hash.format.DefaultHashFormatFactory  
passwordService.hashFormatFactory=$hashFormatFactory  
  
passwordMatcher=org.apache.shiro.authc.credential.PasswordMatcher  
passwordMatcher.passwordService=$passwordService  
  
myRealm=com.github.zhangkaitao.shiro.chapter5.hash.realm.MyRealm  
myRealm.passwordService=$passwordService  
myRealm.credentialsMatcher=$passwordMatcher  
securityManager.realms=$myRealm   
```

2.1、passwordService使用DefaultPasswordService，如果有必要也可以自定义；

2.2、hashService定义散列密码使用的HashService，默认使用DefaultHashService（默认SHA-256算法）；

2.3、hashFormat用于对散列出的值进行格式化，默认使用Shiro1CryptFormat，另外提供了Base64Format和HexFormat，对于有salt的密码请自定义实现ParsableHashFormat然后把salt格式化到散列值中；

2.4、hashFormatFactory用于根据散列值得到散列的密码和salt；因为如果使用如SHA算法，那么会生成一个salt，此salt需要保存到散列后的值中以便之后与传入的密码比较时使用；默认使用DefaultHashFormatFactory；

2.5、passwordMatcher使用PasswordMatcher，其是一个CredentialsMatcher实现；

2.6、将credentialsMatcher赋值给myRealm，myRealm间接继承了AuthenticatingRealm，其在调用getAuthenticationInfo方法获取到AuthenticationInfo信息后，会使用credentialsMatcher来验证凭据是否匹配，如果不匹配将抛出IncorrectCredentialsException异常。



如上方式的缺点是：salt保存在散列值中；没有实现如密码重试次数限制。

**HashedCredentialsMatcher实现密码验证服务**

 Shiro提供了CredentialsMatcher的散列实现HashedCredentialsMatcher，和之前的PasswordMatcher不同的是，它只用于密码验证，且可以提供自己的盐，而不是随机生成盐，且生成密码散列值的算法需要自己写，因为能提供自己的盐。 

**1、生成密码散列值**

 此处我们使用MD5算法，“密码+盐（用户名+随机数）”的方式生成散列值：  

```java
String algorithmName = "md5";  
String username = "liu";  
String password = "123";  
String salt1 = username;  
String salt2 = new SecureRandomNumberGenerator().nextBytes().toHex();  
int hashIterations = 2;  
  
SimpleHash hash = new SimpleHash(algorithmName, password, salt1 + salt2, hashIterations);  
String encodedPassword = hash.toHex(); 
```

如果要写用户模块，需要在新增用户/重置密码时使用如上算法保存密码，将生成的密码及salt2存入数据库（因为我们的散列算法是：md5(md5(密码+username+salt2))）。

 2、生成Realm（com.github.zhangkaitao.shiro.chapter5.hash.realm.MyRealm2） 

```java
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {  
    String username = "liu"; //用户名及salt1  
    String password = "202cb962ac59075b964b07152d234b70"; //加密后的密码  
    String salt2 = "202cb962ac59075b964b07152d234b70";  
SimpleAuthenticationInfo ai =   
        new SimpleAuthenticationInfo(username, password, getName());  
    ai.setCredentialsSalt(ByteSource.Util.bytes(username+salt2)); //盐是用户名+随机数  
        return ai;  
}   
```

此处就是把步骤1中生成的相应数据组装为SimpleAuthenticationInfo，通过SimpleAuthenticationInfo的credentialsSalt设置盐，HashedCredentialsMatcher会自动识别这个盐。

 如果使用JdbcRealm，需要修改获取用户信息（包括盐）的sql：“select password, password_salt from users where username = ?”，而我们的盐是由username+password_salt组成，所以需要通过如下ini配置（shiro-jdbc-hashedCredentialsMatcher.ini）修改：  

```ini
jdbcRealm.saltStyle=COLUMN  
jdbcRealm.authenticationQuery=select password, concat(username,password_salt) from users where username = ?  
jdbcRealm.credentialsMatcher=$credentialsMatcher   
```

1、saltStyle表示使用密码+盐的机制，authenticationQuery第一列是密码，第二列是盐；

2、通过authenticationQuery指定密码及盐查询SQL；



此处还要注意Shiro默认使用了apache commons BeanUtils，默认是不进行Enum类型转型的，此时需要自己注册一个Enum转换器“BeanUtilsBean.getInstance().getConvertUtils().register(new EnumConverter(), JdbcRealm.SaltStyle.class);”具体请参考示例“com.github.zhangkaitao.shiro.chapter5.hash.PasswordTest”中的代码。 



另外可以参考配置shiro-jdbc-passwordservice.ini，提供了JdbcRealm的测试用例，测试前请先调用sql/shiro-init-data.sql初始化用户数据。

 

3、ini配置（shiro-hashedCredentialsMatcher.ini）  

```ini
[main]  
credentialsMatcher=org.apache.shiro.authc.credential.HashedCredentialsMatcher  
credentialsMatcher.hashAlgorithmName=md5  
credentialsMatcher.hashIterations=2  
credentialsMatcher.storedCredentialsHexEncoded=true  
myRealm=com.github.zhangkaitao.shiro.chapter5.hash.realm.MyRealm2  
myRealm.credentialsMatcher=$credentialsMatcher  
securityManager.realms=$myRealm 
```

1、通过credentialsMatcher.hashAlgorithmName=md5指定散列算法为md5，需要和生成密码时的一样；

2、credentialsMatcher.hashIterations=2，散列迭代次数，需要和生成密码时的意义；

3、credentialsMatcher.storedCredentialsHexEncoded=true表示是否存储散列后的密码为16进制，需要和生成密码时的一样，默认是base64；



此处最需要注意的就是HashedCredentialsMatcher的算法需要和生成密码时的算法一样。另外HashedCredentialsMatcher会自动根据AuthenticationInfo的类型是否是SaltedAuthenticationInfo来获取credentialsSalt盐。 

**密码重试次数限制**

如在1个小时内密码最多重试5次，如果尝试次数超过5次就锁定1小时，1小时后可再次重试，如果还是重试失败，可以锁定如1天，以此类推，防止密码被暴力破解。我们通过继承HashedCredentialsMatcher，且使用Ehcache记录重试次数和超时时间。

 ```java
public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {  
       String username = (String)token.getPrincipal();  
        //retry count + 1  
        Element element = passwordRetryCache.get(username);  
        if(element == null) {  
            element = new Element(username , new AtomicInteger(0));  
            passwordRetryCache.put(element);  
        }  
        AtomicInteger retryCount = (AtomicInteger)element.getObjectValue();  
        if(retryCount.incrementAndGet() > 5) {  
            //if retry count > 5 throw  
            throw new ExcessiveAttemptsException();  
        }  
  
        boolean matches = super.doCredentialsMatch(token, info);  
        if(matches) {  
            //clear retry count  
            passwordRetryCache.remove(username);  
        }  
        return matches;  
}   
 ```

如上代码逻辑比较简单，即如果密码输入正确清除cache中的记录；否则cache中的重试次数+1，如果超出5次那么抛出异常表示超出重试次数了。

###  Realm及相关对象

####  Realm

接下来看下一般真实环境下的Realm如何实现。

**1、定义实体及关系**

 ![](http://dl2.iteye.com/upload/attachment/0094/1329/dd9f6a00-f6bc-3563-8afd-0c11048060b8.png)



上图表示  用户-角色之间是多对多关系，角色-权限之间是多对多关系；且用户和权限之间通过角色建立关系；在系统中验证时通过权限验证，角色只是权限集合，即所谓的显示角色；其实权限应该对应到资源（如菜单、URL、页面按钮、Java方法等）中，即应该将权限字符串存储到资源实体中，但是目前为了简单化，直接提取一个权限表。

用户实体包括：编号(id)、用户名(username)、密码(password)、盐(salt)、是否锁定(locked)；是否锁定用于封禁用户使用，其实最好使用Enum字段存储，可以实现更复杂的用户状态实现。

角色实体包括：、编号(id)、角色标识符（role）、描述（description）、是否可用（available）；其中角色标识符用于在程序中进行隐式角色判断的，描述用于以后再前台界面显示的、是否可用表示角色当前是否激活。

权限实体包括：编号（id）、权限标识符（permission）、描述（description）、是否可用（available）；含义和角色实体类似不再阐述。



另外还有两个关系实体：用户-角色实体（用户编号、角色编号，且组合为复合主键）；角色-权限实体（角色编号、权限编号，且组合为复合主键）。

 **2、环境准备**

 为了方便数据库操作，使用了“org.springframework: spring-jdbc: 4.0.0.RELEASE”依赖，虽然是spring4版本的，但使用上和spring3无区别。其他依赖请参考源码的pom.xml。

 **3、定义Service及Dao**

 为了实现的简单性，只实现必须的功能，其他的可以自己实现即可。

 **PermissionService**  

```java
public interface PermissionService {  
    public Permission createPermission(Permission permission);  
    public void deletePermission(Long permissionId);  
}  
```

实现基本的创建/删除权限。

 **RoleService**  

```java
public interface RoleService {  
    public Role createRole(Role role);  
    public void deleteRole(Long roleId);  
    //添加角色-权限之间关系  
    public void correlationPermissions(Long roleId, Long... permissionIds);  
    //移除角色-权限之间关系  
    public void uncorrelationPermissions(Long roleId, Long... permissionIds);//  
}   
```

相对于PermissionService多了关联/移除关联角色-权限功能。

 **UserService**  

```java
public interface UserService {  
    public User createUser(User user); //创建账户  
    public void changePassword(Long userId, String newPassword);//修改密码  
    public void correlationRoles(Long userId, Long... roleIds); //添加用户-角色关系  
    public void uncorrelationRoles(Long userId, Long... roleIds);// 移除用户-角色关系  
    public User findByUsername(String username);// 根据用户名查找用户  
    public Set<String> findRoles(String username);// 根据用户名查找其角色  
    public Set<String> findPermissions(String username); //根据用户名查找其权限  
}
```

此处使用findByUsername、findRoles及findPermissions来查找用户名对应的帐号、角色及权限信息。之后的Realm就使用这些方法来查找相关信息。 

**UserServiceImpl**   

```java
public User createUser(User user) {  
    //加密密码  
    passwordHelper.encryptPassword(user);  
    return userDao.createUser(user);  
}  
public void changePassword(Long userId, String newPassword) {  
    User user =userDao.findOne(userId);  
    user.setPassword(newPassword);  
    passwordHelper.encryptPassword(user);  
    userDao.updateUser(user);  
}
```

在创建账户及修改密码时直接把生成密码操作委托给PasswordHelper。

 **PasswordHelper** 

```java
public class PasswordHelper {  
    private RandomNumberGenerator randomNumberGenerator =  
     new SecureRandomNumberGenerator();  
    private String algorithmName = "md5";  
    private final int hashIterations = 2;  
    public void encryptPassword(User user) {  
        user.setSalt(randomNumberGenerator.nextBytes().toHex());  
        String newPassword = new SimpleHash(  
                algorithmName,  
                user.getPassword(),  
                ByteSource.Util.bytes(user.getCredentialsSalt()),  
                hashIterations).toHex();  
        user.setPassword(newPassword);  
    }  
}  
```

之后的CredentialsMatcher需要和此处加密的算法一样。user.getCredentialsSalt()辅助方法返回username+salt。

 

为了节省篇幅，对于DAO/Service的接口及实现，具体请参考源码com.github.zhangkaitao.shiro.chapter6。另外请参考Service层的测试用例com.github.zhangkaitao.shiro.chapter6.service.ServiceTest。

**4、定义Realm**

 **RetryLimitHashedCredentialsMatcher**  

**UserRealm** 

```java
public class UserRealm extends AuthorizingRealm {  
    private UserService userService = new UserServiceImpl();  
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {  
        String username = (String)principals.getPrimaryPrincipal();  
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();  
        authorizationInfo.setRoles(userService.findRoles(username));  
        authorizationInfo.setStringPermissions(userService.findPermissions(username));  
        return authorizationInfo;  
    }  
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {  
        String username = (String)token.getPrincipal();  
        User user = userService.findByUsername(username);  
        if(user == null) {  
            throw new UnknownAccountException();//没找到帐号  
        }  
        if(Boolean.TRUE.equals(user.getLocked())) {  
            throw new LockedAccountException(); //帐号锁定  
        }  
        //交给AuthenticatingRealm使用CredentialsMatcher进行密码匹配，如果觉得人家的不好可以在此判断或自定义实现  
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(  
                user.getUsername(), //用户名  
                user.getPassword(), //密码  
                ByteSource.Util.bytes(user.getCredentialsSalt()),//salt=username+salt  
                getName()  //realm name  
        );  
        return authenticationInfo;  
    }  
}   
```

**1、UserRealm父类AuthorizingRealm将获取Subject相关信息分成两步**：获取身份验证信息（doGetAuthenticationInfo）及授权信息（doGetAuthorizationInfo）；

**2、doGetAuthenticationInfo获取身份验证相关信息**：首先根据传入的用户名获取User信息；然后如果user为空，那么抛出没找到帐号异常UnknownAccountException；如果user找到但锁定了抛出锁定异常LockedAccountException；最后生成AuthenticationInfo信息，交给间接父类AuthenticatingRealm使用CredentialsMatcher进行判断密码是否匹配，如果不匹配将抛出密码错误异常IncorrectCredentialsException；另外如果密码重试此处太多将抛出超出重试次数异常ExcessiveAttemptsException；在组装SimpleAuthenticationInfo信息时，需要传入：身份信息（用户名）、凭据（密文密码）、盐（username+salt），CredentialsMatcher使用盐加密传入的明文密码和此处的密文密码进行匹配。

**3、doGetAuthorizationInfo获取授权信息**：PrincipalCollection是一个身份集合，因为我们现在就一个Realm，所以直接调用getPrimaryPrincipal得到之前传入的用户名即可；然后根据用户名调用UserService接口获取角色及权限信息。

**5、测试用例**

为了节省篇幅，请参考测试用例com.github.zhangkaitao.shiro.chapter6.realm.UserRealmTest。包含了：登录成功、用户名错误、密码错误、密码超出重试次数、有/没有角色、有/没有权限的测试。

#### AuthenticationToken

![](http://dl2.iteye.com/upload/attachment/0094/1333/6c026012-2583-3a26-af70-bb1b0eae491b.png)

AuthenticationToken用于收集用户提交的身份（如用户名）及凭据（如密码）：  

```java
public interface AuthenticationToken extends Serializable {  
    Object getPrincipal(); //身份  
    Object getCredentials(); //凭据  
}   
```

扩展接口RememberMeAuthenticationToken：提供了“boolean isRememberMe()”现“记住我”的功能；

扩展接口是HostAuthenticationToken：提供了“String getHost()”方法用于获取用户“主机”的功能。

 

Shiro提供了一个直接拿来用的UsernamePasswordToken，用于实现用户名/密码Token组，另外其实现了RememberMeAuthenticationToken和HostAuthenticationToken，可以实现记住我及主机验证的支持。

#### AuthenticationInfo

![](http://dl2.iteye.com/upload/attachment/0094/1337/0be597af-cd61-34ce-b1f0-8ebd15dbdeb9.png)

AuthenticationInfo有两个作用：

 1、如果Realm是AuthenticatingRealm子类，则提供给AuthenticatingRealm内部使用的CredentialsMatcher进行凭据验证；（如果没有继承它需要在自己的Realm中自己实现验证）；

 2、提供给SecurityManager来创建Subject（提供身份信息）；



 MergableAuthenticationInfo用于提供在多Realm时合并AuthenticationInfo的功能，主要合并Principal、如果是其他的如credentialsSalt，会用后边的信息覆盖前边的。

 

比如HashedCredentialsMatcher，在验证时会判断AuthenticationInfo是否是SaltedAuthenticationInfo子类，来获取盐信息。

 

Account相当于我们之前的User，SimpleAccount是其一个实现；在IniRealm、PropertiesRealm这种静态创建帐号信息的场景中使用，这些Realm直接继承了SimpleAccountRealm，而SimpleAccountRealm提供了相关的API来动态维护SimpleAccount；即可以通过这些API来动态增删改查SimpleAccount；动态增删改查角色/权限信息。及如果您的帐号不是特别多，可以使用这种方式，具体请参考SimpleAccountRealm Javadoc。

#### PrincipalCollection

 ![](http://dl2.iteye.com/upload/attachment/0094/1341/772c988b-e930-31d1-ad14-c9ae8f476f66.png)

因为我们可以在Shiro中同时配置多个Realm，所以呢身份信息可能就有多个；因此其提供了PrincipalCollection 用于聚合这些身份信息：

```java
public interface PrincipalCollection extends Iterable, Serializable {  
    Object getPrimaryPrincipal(); //得到主要的身份  
    <T> T oneByType(Class<T> type); //根据身份类型获取第一个  
    <T> Collection<T> byType(Class<T> type); //根据身份类型获取一组  
    List asList(); //转换为List  
    Set asSet(); //转换为Set  
    Collection fromRealm(String realmName); //根据Realm名字获取  
    Set<String> getRealmNames(); //获取所有身份验证通过的Realm名字  
    boolean isEmpty(); //判断是否为空  
}  
```

因为PrincipalCollection聚合了多个，此处最需要注意的是getPrimaryPrincipal，如果只有一个Principal那么直接返回即可，如果有多个Principal，则返回第一个（因为内部使用Map存储，所以可以认为是返回任意一个）；oneByType / byType根据凭据的类型返回相应的Principal；fromRealm根据Realm名字（每个Principal都与一个Realm关联）获取相应的Principal。

 

MutablePrincipalCollection是一个可变的PrincipalCollection接口，即提供了如下可变方法：  

```java
public interface MutablePrincipalCollection extends PrincipalCollection {  
    void add(Object principal, String realmName); //添加Realm-Principal的关联  
    void addAll(Collection principals, String realmName); //添加一组Realm-Principal的关联  
    void addAll(PrincipalCollection principals);//添加PrincipalCollection  
    void clear();//清空  
}   
```

目前Shiro只提供了一个实现SimplePrincipalCollection，还记得之前的AuthenticationStrategy实现嘛，用于在多Realm时判断是否满足条件的，在大多数实现中（继承了AbstractAuthenticationStrategy）afterAttempt方法会进行AuthenticationInfo（实现了MergableAuthenticationInfo）的merge，比如SimpleAuthenticationInfo会合并多个Principal为一个PrincipalCollection。

 

对于PrincipalMap是Shiro 1.2中的一个实验品，暂时无用，具体可以参考其Javadoc。接下来通过示例来看看PrincipalCollection。

 

**1、准备三个Realm**

 **MyRealm1** 

```java
public class MyRealm1 implements Realm {  
    @Override  
    public String getName() {  
        return "a"; //realm name 为 “a”  
    }  
    //省略supports方法，具体请见源码  
    @Override  
    public AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {  
        return new SimpleAuthenticationInfo(  
                "zhang", //身份 字符串类型  
                "123",   //凭据  
                getName() //Realm Name  
        );  
    }  
}  
```

**MyRealm2**  

和MyRealm1完全一样，只是Realm名字为b。

 **MyRealm3** 

```java
public class MyRealm3 implements Realm {  
    @Override  
    public String getName() {  
        return "c"; //realm name 为 “c”  
    }  
    //省略supports方法，具体请见源码  
    @Override  
    public AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {  
        User user = new User("zhang", "123");  
        return new SimpleAuthenticationInfo(  
                user, //身份 User类型  
                "123",   //凭据  
                getName() //Realm Name  
        );  
    }  
}   
```

和MyRealm1同名，但返回的Principal是User类型。

#### AuthorizationInfo

![](http://dl2.iteye.com/upload/attachment/0094/1345/c8868cea-6b01-34cc-b7be-45ee244bd217.png)

AuthorizationInfo用于聚合授权信息的：  

```java
public interface AuthorizationInfo extends Serializable {  
    Collection<String> getRoles(); //获取角色字符串信息  
    Collection<String> getStringPermissions(); //获取权限字符串信息  
    Collection<Permission> getObjectPermissions(); //获取Permission对象信息  
}  
```

当我们使用AuthorizingRealm时，如果身份验证成功，在进行授权时就通过doGetAuthorizationInfo方法获取角色/权限信息用于授权验证。



Shiro提供了一个实现SimpleAuthorizationInfo，大多数时候使用这个即可。



 对于Account及SimpleAccount，之前的【6.3 AuthenticationInfo】已经介绍过了，用于SimpleAccountRealm子类，实现动态角色/权限维护的。

####  Subject

 ![](http://dl2.iteye.com/upload/attachment/0094/1347/131b0c95-9aed-36b0-9c09-0a6f9c6b3605.png)

Subject是Shiro的核心对象，基本所有身份验证、授权都是通过Subject完成。

 **1、身份信息获取**  

```java
Object getPrincipal(); //Primary Principal  
PrincipalCollection getPrincipals(); // PrincipalCollection  
```

**2、身份验证**  

```java
void login(AuthenticationToken token) throws AuthenticationException;  
boolean isAuthenticated();  
boolean isRemembered();  
```

通过login登录，如果登录失败将抛出相应的AuthenticationException，如果登录成功调用isAuthenticated就会返回true，即已经通过身份验证；如果isRemembered返回true，表示是通过记住我功能登录的而不是调用login方法登录的。isAuthenticated/isRemembered是互斥的，即如果其中一个返回true，另一个返回false。

 **3、角色授权验证**   

```java
boolean hasRole(String roleIdentifier);  
boolean[] hasRoles(List<String> roleIdentifiers);  
boolean hasAllRoles(Collection<String> roleIdentifiers);  
void checkRole(String roleIdentifier) throws AuthorizationException;  
void checkRoles(Collection<String> roleIdentifiers) throws AuthorizationException;  
void checkRoles(String... roleIdentifiers) throws AuthorizationException; 
```

hasRole*进行角色验证，验证后返回true/false；而checkRole*验证失败时抛出AuthorizationException异常。 

 **4、权限授权验证**  

```java
boolean isPermitted(String permission);  
boolean isPermitted(Permission permission);  
boolean[] isPermitted(String... permissions);  
boolean[] isPermitted(List<Permission> permissions);  
boolean isPermittedAll(String... permissions);  
boolean isPermittedAll(Collection<Permission> permissions);  
void checkPermission(String permission) throws AuthorizationException;  
void checkPermission(Permission permission) throws AuthorizationException;  
void checkPermissions(String... permissions) throws AuthorizationException;  
void checkPermissions(Collection<Permission> permissions) throws AuthorizationException; 
```

isPermitted*进行权限验证，验证后返回true/false；而checkPermission*验证失败时抛出AuthorizationException。

 **5、会话**  

```java
Session getSession(); //相当于getSession(true)  
Session getSession(boolean create);  
```

类似于Web中的会话。如果登录成功就相当于建立了会话，接着可以使用getSession获取；如果create=false如果没有会话将返回null，而create=true如果没有会话会强制创建一个。

 **6、退出**   

```java
void logout();  
```

**7、RunAs**    

```java
void runAs(PrincipalCollection principals) throws NullPointerException, IllegalStateException;  
boolean isRunAs();  
PrincipalCollection getPreviousPrincipals();  
PrincipalCollection releaseRunAs();   
```

RunAs即实现“允许A假设为B身份进行访问”；通过调用subject.runAs(b)进行访问；接着调用subject.getPrincipals将获取到B的身份；此时调用isRunAs将返回true；而a的身份需要通过subject. getPreviousPrincipals获取；如果不需要RunAs了调用subject. releaseRunAs即可。

 **8、多线程**  

```java
<V> V execute(Callable<V> callable) throws ExecutionException;  
void execute(Runnable runnable);  
<V> Callable<V> associateWith(Callable<V> callable);  
Runnable associateWith(Runnable runnable);   
```

实现线程之间的Subject传播，因为Subject是线程绑定的；因此在多线程执行中需要传播到相应的线程才能获取到相应的Subject。最简单的办法就是通过execute(runnable/callable实例)直接调用；或者通过associateWith(runnable/callable实例)得到一个包装后的实例；它们都是通过：1、把当前线程的Subject绑定过去；2、在线程执行结束后自动释放。 



Subject自己不会实现相应的身份验证/授权逻辑，而是通过DelegatingSubject委托给SecurityManager实现；及可以理解为Subject是一个面门。

 

对于Subject的构建一般没必要我们去创建；一般通过SecurityUtils.getSubject()获取：  

```java
public static Subject getSubject() {  
    Subject subject = ThreadContext.getSubject();  
    if (subject == null) {  
        subject = (new Subject.Builder()).buildSubject();  
        ThreadContext.bind(subject);  
    }  
    return subject;  
}   
```

即首先查看当前线程是否绑定了Subject，如果没有通过Subject.Builder构建一个然后绑定到现场返回。

 如果想自定义创建，可以通过：  

```java
new Subject.Builder().principals(身份).authenticated(true/false).buildSubject()  
```

这种可以创建相应的Subject实例了，然后自己绑定到线程即可。在new Builder()时如果没有传入SecurityManager，自动调用SecurityUtils.getSecurityManager获取；也可以自己传入一个实例。

 对于Subject我们一般这么使用：

**1.身份验证（login）**

**2.授权（hasRole\*/isPermitted*或checkRole*/checkPermission*）**

**3.将相应的数据存储到会话（Session）**

**4.切换身份（RunAs）/多线程身份传播**

**5.退出**

而我们必须的功能就是1、2、5。到目前为止我们就可以使用Shiro进行应用程序的安全控制了，但是还是缺少如对Web验证、Java方法验证等的一些简化实现。      

### 与Web集成

Shiro提供了与Web集成的支持，其通过一个ShiroFilter入口来拦截需要安全控制的URL，然后进行相应的控制，ShiroFilter类似于如Strut2/SpringMVC这种web框架的前端控制器，其是安全控制的入口点，其负责读取配置（如ini配置文件），然后判断URL是否需要登录/权限等工作。

####  准备环境

**创建webapp应用**  

此处我们使用了jetty-maven-plugin和tomcat7-maven-plugin插件；这样可以直接使用“mvn jetty:run”或“mvn tomcat7:run”直接运行webapp了。然后通过URLhttp://localhost:8080/chapter7/访问即可。

 shiro-web  

```xml
<dependency>  
    <groupId>org.apache.shiro</groupId>  
    <artifactId>shiro-web</artifactId>  
    <version>1.2.2</version>  
</dependency>   
```

#### ShiroFilter入口

**1、Shiro 1.1及以前版本配置方式**    

```xml
<filter>  
    <filter-name>iniShiroFilter</filter-name>  
    <filter-class>org.apache.shiro.web.servlet.IniShiroFilter</filter-class>  
    <init-param>  
        <param-name>configPath</param-name>  
        <param-value>classpath:shiro.ini</param-value>  
    </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>iniShiroFilter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>   
```

1、使用IniShiroFilter作为Shiro安全控制的入口点，通过url-pattern指定需要安全的URL；

2、通过configPath指定ini配置文件位置，默认是先从/WEB-INF/shiro.ini加载，如果没有就默认加载classpath:shiro.ini，即默认相对于web应用上下文根路径；

3、也可以通过如下方式直接内嵌ini配置文件内容到web.xml

```xml
<init-param>  
    <param-name>config</param-name>  
    <param-value>  
        ini配置文件贴在这  
    </param-value>  
</init-param>  
```

**2、Shiro 1.2及以后版本的配置方式**

从Shiro 1.2开始引入了Environment/WebEnvironment的概念，即由它们的实现提供相应的SecurityManager及其相应的依赖。ShiroFilter会自动找到Environment然后获取相应的依赖。

```xml
<listener>  
   <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>  
</listener>  
```

通过EnvironmentLoaderListener来创建相应的WebEnvironment，并自动绑定到ServletContext，默认使用IniWebEnvironment实现。

 可以通过如下配置修改默认实现及其加载的配置文件位置：  

```xml
<context-param>  
   <param-name>shiroEnvironmentClass</param-name>  
   <param-value>org.apache.shiro.web.env.IniWebEnvironment</param-value>  
</context-param>  
    <context-param>  
        <param-name>shiroConfigLocations</param-name>  
        <param-value>classpath:shiro.ini</param-value>  
    </context-param>   
```

shiroConfigLocations默认是“/WEB-INF/shiro.ini”，IniWebEnvironment默认是先从/WEB-INF/shiro.ini加载，如果没有就默认加载classpath:shiro.ini。

 **3、与Spring集成**  

```xml
<filter>  
    <filter-name>shiroFilter</filter-name>  
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
    <init-param>  
        <param-name>targetFilterLifecycle</param-name>  
        <param-value>true</param-value>  
    </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>shiroFilter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>  
```

DelegatingFilterProxy作用是自动到spring容器查找名字为shiroFilter（filter-name）的bean并把所有Filter的操作委托给它。然后将ShiroFilter配置到spring容器即可：  

```xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">  
<property name="securityManager" ref="securityManager"/>  
<!—忽略其他，详见与Spring集成部分 -->  
</bean>   
```

最后不要忘了使用org.springframework.web.context.ContextLoaderListener加载这个spring配置文件即可。放在struts或者springmvc的前端控制器上面。

因为我们现在的shiro版本是1.2的，因此之后的测试都是使用1.2的配置。

#### Web INI配置

ini配置部分和之前的相比将多出对url部分的配置。       

```ini
[main]  
#默认是/login.jsp  
authc.loginUrl=/login  
roles.unauthorizedUrl=/unauthorized  
perms.unauthorizedUrl=/unauthorized  
[users]  
zhang=123,admin  
wang=123  
[roles]  
admin=user:*,menu:*  
[urls]  
/login=anon  
/unauthorized=anon  
/static/**=anon  
/authenticated=authc  
/role=authc,roles[admin]  
/permission=authc,perms["user:create"]  
```

其中最重要的就是[urls]部分的配置，其格式是： “url=拦截器[参数]，拦截器[参数]”；即如果当前请求的url匹配[urls]部分的某个url模式，将会执行其配置的拦截器。比如anon拦截器表示匿名访问（即不需要登录即可访问）；authc拦截器表示需要身份认证通过后才能访问；roles[admin]拦截器表示需要有admin角色授权才能访问；而perms["user:create"]拦截器表示需要有“user:create”权限才能访问。

 

**url****模式使用Ant风格模式**

Ant路径通配符支持?、*、**，注意通配符匹配不包括目录分隔符“/”：

**?****：匹配一个字符**，如”/admin?”将匹配/admin1，但不匹配/admin或/admin2；

*******：匹配零个或多个字符串**，如/admin*将匹配/admin、/admin123，但不匹配/admin/1；

***\*****：匹配路径中的零个或多个路径**，如/admin/**将匹配/admin/a或/admin/a/b。



**url****模式匹配顺序**

url模式匹配顺序是按照在配置中的声明顺序匹配，即从头开始使用第一个匹配的url模式对应的拦截器链。如：

```ini
/bb/**=filter1  
/bb/aa=filter2  
/**=filter3 
```

如果请求的url是“/bb/aa”，因为按照声明顺序进行匹配，那么将使用filter1进行拦截。

 

拦截器将在下一节详细介绍。接着我们来看看身份验证、授权及退出在web中如何实现。

 **1、身份验证（登录）**

**1.1、首先配置需要身份验证的url**  

```ini
/authenticated=authc  
/role=authc,roles[admin]  
/permission=authc,perms["user:create"]
```

即访问这些地址时会首先判断用户有没有登录，如果没有登录默会跳转到登录页面，默认是/login.jsp，可以通过在[main]部分通过如下配置修改：   

```ini
authc.loginUrl=/login  
```

**1.2、登录Servle** 

```java
@WebServlet(name = "loginServlet", urlPatterns = "/login")  
public class LoginServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        req.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(req, resp);  
    }  
    @Override  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        String error = null;  
        String username = req.getParameter("username");  
        String password = req.getParameter("password");  
        Subject subject = SecurityUtils.getSubject();  
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);  
        try {  
            subject.login(token);  
        } catch (UnknownAccountException e) {  
            error = "用户名/密码错误";  
        } catch (IncorrectCredentialsException e) {  
            error = "用户名/密码错误";  
        } catch (AuthenticationException e) {  
            //其他错误，比如锁定，如果想单独处理请单独catch处理  
            error = "其他错误：" + e.getMessage();  
        }  
        if(error != null) {//出错了，返回登录页面  
            req.setAttribute("error", error);  
            req.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(req, resp);  
        } else {//登录成功  
            req.getRequestDispatcher("/WEB-INF/jsp/loginSuccess.jsp").forward(req, resp);  
        }  
    }  
}  
```

1、doGet请求时展示登录页面；

2、doPost时进行登录，登录时收集username/password参数，然后提交给Subject进行登录。如果有错误再返回到登录页面；否则跳转到登录成功页面（此处应该返回到访问登录页面之前的那个页面，或者没有上一个页面时访问主页）。

3、JSP页面请参考源码。

**1.3****、测试**

首先输入http://localhost:8080/chapter7/login进行登录，登录成功后接着可以访问http://localhost:8080/chapter7/authenticated来显示当前登录的用户： 

1. ${subject.principal}身份验证已通过。  

当前实现的一个缺点就是，永远返回到同一个成功页面（比如首页），在实际项目中比如支付时如果没有登录将跳转到登录页面，登录成功后再跳回到支付页面；对于这种功能大家可以在登录时把当前请求保存下来，然后登录成功后再重定向到该请求即可。



Shiro内置了登录（身份验证）的实现：基于表单的和基于Basic的验证，其通过拦截器实现。

 

#### **2、基于Basic的拦截器身份验证** 

**2.1、shiro-basicfilterlogin.ini配置**   

```ini
[main]  
authcBasic.applicationName=please login  
………省略users  
[urls]  
/role=authcBasic,roles[admin]  
```

1、authcBasic是org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter类型的实例，其用于实现基于Basic的身份验证；applicationName用于弹出的登录框显示信息使用，如图：

 ![](http://dl2.iteye.com/upload/attachment/0094/3895/00e16ed3-afea-3614-926c-bdc2b09d5d8e.png)

2、[urls]部分配置了/role地址需要走authcBasic拦截器，即如果访问/role时还没有通过身份验证那么将弹出如上图的对话框进行登录，登录成功即可访问。

 **2.2、web.xml**

把shiroConfigLocations改为shiro-basicfilterlogin.ini即可

**2.3、测试**

输入http://localhost:8080/chapter7/role，会弹出之前的Basic验证对话框输入“zhang/123”即可登录成功进行访问。

#### **3、基于表单的拦截器身份验证**

基于表单的拦截器身份验证和【1】类似，但是更简单，因为其已经实现了大部分登录逻辑；我们只需要指定：登录地址/登录失败后错误信息存哪/成功的地址即可。

**3.1、shiro-formfilterlogin.ini**   

```ini
[main]  
authc.loginUrl=/formfilterlogin  
authc.usernameParam=username  
authc.passwordParam=password  
authc.successUrl=/  
authc.failureKeyAttribute=shiroLoginFailure  
  
[urls]  
/role=authc,roles[admin]
```

1、authc是org.apache.shiro.web.filter.authc.FormAuthenticationFilter类型的实例，其用于实现基于表单的身份验证；通过loginUrl指定当身份验证时的登录表单；usernameParam指定登录表单提交的用户名参数名；passwordParam指定登录表单提交的密码参数名；successUrl指定登录成功后重定向的默认地址（默认是“/”）（如果有上一个地址会自动重定向带该地址）；failureKeyAttribute指定登录失败时的request属性key（默认shiroLoginFailure）；这样可以在登录表单得到该错误key显示相应的错误消息；

 **3.2、web.xml**

把shiroConfigLocations改为shiro- formfilterlogin.ini即可。

**3.3、登录Servlet**   

```java
@WebServlet(name = "formFilterLoginServlet", urlPatterns = "/formfilterlogin")  
public class FormFilterLoginServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        doPost(req, resp);  
    }  
    @Override  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)  
     throws ServletException, IOException {  
        String errorClassName = (String)req.getAttribute("shiroLoginFailure");  
        if(UnknownAccountException.class.getName().equals(errorClassName)) {  
            req.setAttribute("error", "用户名/密码错误");  
        } else if(IncorrectCredentialsException.class.getName().equals(errorClassName)) {  
            req.setAttribute("error", "用户名/密码错误");  
        } else if(errorClassName != null) {  
            req.setAttribute("error", "未知错误：" + errorClassName);  
        }  
        req.getRequestDispatcher("/WEB-INF/jsp/formfilterlogin.jsp").forward(req, resp);  
    }  
}
```

在登录Servlet中通过shiroLoginFailure得到authc登录失败时的异常类型名，然后根据此异常名来决定显示什么错误消息。

 **4、测试**

 输入http://localhost:8080/chapter7/role，会跳转到“/formfilterlogin”登录表单，提交表单如果authc拦截器登录成功后，会直接重定向会之前的地址“/role”；假设我们直接访问“/formfilterlogin”的话登录成功将直接到默认的successUrl。

#### 授权（角色/权限验证）

 **4.1、shiro.ini**     

```ini
[main]  
roles.unauthorizedUrl=/unauthorized  
perms.unauthorizedUrl=/unauthorized  
 [urls]  
/role=authc,roles[admin]  
/permission=authc,perms["user:create"] 
```

通过unauthorizedUrl属性指定如果授权失败时重定向到的地址。roles是org.apache.shiro.web.filter.authz.RolesAuthorizationFilter类型的实例，通过参数指定访问时需要的角色，如“[admin]”，如果有多个使用“，”分割，且验证时是hasAllRole验证，即且的关系。Perms是org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter类型的实例，和roles类似，只是验证权限字符串。

 **4.2、web.xml**

 把shiroConfigLocations改为shiro.ini即可。

####  **4.3、RoleServlet/PermissionServlet**    

```java
@WebServlet(name = "permissionServlet", urlPatterns = "/permission")  
public class PermissionServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        Subject subject = SecurityUtils.getSubject();  
        subject.checkPermission("user:create");  
        req.getRequestDispatcher("/WEB-INF/jsp/hasPermission.jsp").forward(req, resp);  
    }  
}  
```

```java
@WebServlet(name = "roleServlet", urlPatterns = "/role")  
public class RoleServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        Subject subject = SecurityUtils.getSubject();  
        subject.checkRole("admin");  
        req.getRequestDispatcher("/WEB-INF/jsp/hasRole.jsp").forward(req, resp);  
    }  
}  
```

**4.4、测试**

 首先访问http://localhost:8080/chapter7/login，使用帐号“zhang/123”进行登录，再访问/role或/permission时会跳转到成功页面（因为其授权成功了）；如果使用帐号“wang/123”登录成功后访问这两个地址会跳转到“/unauthorized”即没有授权页面。 

#### **退出** 

**5.1、shiro.ini**   

```ini
[urls]  
/logout=anon  
```

**5.2、LogoutServlet**  

```java
@WebServlet(name = "logoutServlet", urlPatterns = "/logout")  
public class LogoutServlet extends HttpServlet {  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
      throws ServletException, IOException {  
        SecurityUtils.getSubject().logout();  
        req.getRequestDispatcher("/WEB-INF/jsp/logoutSuccess.jsp").forward(req, resp);  
    }  
} 
```

直接调用Subject.logout即可，退出成功后转发/重定向到相应页面即可。

 **5.3、测试**

首先访问http://localhost:8080/chapter7/login，使用帐号“zhang/123”进行登录，登录成功后访问/logout即可退出。

 

Shiro也提供了logout拦截器用于退出，其是org.apache.shiro.web.filter.authc.LogoutFilter类型的实例，我们可以在shiro.ini配置文件中通过如下配置完成退出：

```ini
[main]  
logout.redirectUrl=/login  
  
[urls]  
/logout2=logout  
```

通过logout.redirectUrl指定退出后重定向的地址；通过/logout2=logout指定退出url是/logout2。这样当我们登录成功后然后访问/logout2即可退出。

###  拦截器机制

#### 拦截器介绍

Shiro使用了与Servlet一样的Filter接口进行扩展；所以如果对Filter不熟悉可以参考《Servlet3.1规范》http://www.iteye.com/blogs/subjects/Servlet-3-1 了解Filter的工作原理。首先下图是Shiro拦截器的基础类图：

![](http://dl2.iteye.com/upload/attachment/0094/3897/b910abb9-2ef0-33b7-af4d-4c645263ed54.png)



**1、NameableFilter**

NameableFilter给Filter起个名字，如果没有设置默认就是FilterName；还记得之前的如authc吗？当我们组装拦截器链时会根据这个名字找到相应的拦截器实例； 

**2、OncePerRequestFilter**

OncePerRequestFilter用于防止多次执行Filter的；也就是说一次请求只会走一次拦截器链；另外提供enabled属性，表示是否开启该拦截器实例，默认enabled=true表示开启，如果不想让某个拦截器工作，可以设置为false即可。

 **3、ShiroFilter** 

ShiroFilter是整个Shiro的入口点，用于拦截需要安全控制的请求进行处理，这个之前已经用过了。

 **4、AdviceFilter**

 AdviceFilter提供了AOP风格的支持，类似于SpringMVC中的Interceptor：  

```java
boolean preHandle(ServletRequest request, ServletResponse response) throws Exception  
void postHandle(ServletRequest request, ServletResponse response) throws Exception  
void afterCompletion(ServletRequest request, ServletResponse response, Exception exception) throws Exception;   
```

preHandler：类似于AOP中的前置增强；在拦截器链执行之前执行；如果返回true则继续拦截器链；否则中断后续的拦截器链的执行直接返回；进行预处理（如基于表单的身份验证、授权）

postHandle：类似于AOP中的后置返回增强；在拦截器链执行完成后执行；进行后处理（如记录执行时间之类的）；

afterCompletion：类似于AOP中的后置最终增强；即不管有没有异常都会执行；可以进行清理资源（如接触Subject与线程的绑定之类的）；

**5、PathMatchingFilter**

 PathMatchingFilter提供了基于Ant风格的请求路径匹配功能及拦截器参数解析的功能，如“roles[admin,user]”自动根据“，”分割解析到一个路径参数配置并绑定到相应的路径：  

```java
boolean pathsMatch(String path, ServletRequest request)  
boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception   
```

pathsMatch：该方法用于path与请求路径进行匹配的方法；如果匹配返回true；

onPreHandle：在preHandle中，当pathsMatch匹配一个路径后，会调用opPreHandler方法并将路径绑定参数配置传给mappedValue；然后可以在这个方法中进行一些验证（如角色授权），如果验证失败可以返回false中断流程；默认返回true；也就是说子类可以只实现onPreHandle即可，无须实现preHandle。如果没有path与请求路径匹配，默认是通过的（即preHandle返回true）。

**6、AccessControlFilter**

 AccessControlFilter提供了访问控制的基础功能；比如是否允许访问/当访问拒绝时如何处理等：  

```java
abstract boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception;  
boolean onAccessDenied(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception;  
abstract boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception;  
```

isAccessAllowed：表示是否允许访问；mappedValue就是[urls]配置中拦截器参数部分，如果允许访问返回true，否则false；

onAccessDenied：表示当访问拒绝时是否已经处理了；如果返回true表示需要继续处理；如果返回false表示该拦截器实例已经处理了，将直接返回即可



onPreHandle会自动调用这两个方法决定是否继续处理：  

```java
boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {  
    return isAccessAllowed(request, response, mappedValue) || onAccessDenied(request, response, mappedValue);  
}   
```

另外AccessControlFilter还提供了如下方法用于处理如登录成功后/重定向到上一个请求：   

```java
void setLoginUrl(String loginUrl) //身份验证时使用，默认/login.jsp  
String getLoginUrl()  
Subject getSubject(ServletRequest request, ServletResponse response) //获取Subject实例  
boolean isLoginRequest(ServletRequest request, ServletResponse response)//当前请求是否是登录请求  
void saveRequestAndRedirectToLogin(ServletRequest request, ServletResponse response) throws IOException //将当前请求保存起来并重定向到登录页面  
void saveRequest(ServletRequest request) //将请求保存起来，如登录成功后再重定向回该请求  
void redirectToLogin(ServletRequest request, ServletResponse response) //重定向到登录页面 
```

比如基于表单的身份验证就需要使用这些功能。

 

到此基本的拦截器就完事了，如果我们想进行访问访问的控制就可以继承AccessControlFilter；如果我们要添加一些通用数据我们可以直接继承PathMatchingFilter。

#### 拦截器链

**Shiro对Servlet容器的FilterChain进行了代理**，即ShiroFilter在继续Servlet容器的Filter链的执行之前，通过ProxiedFilterChain对Servlet容器的FilterChain进行了代理；即先走Shiro自己的Filter体系，然后才会委托给Servlet容器的FilterChain进行Servlet容器级别的Filter链执行；Shiro的ProxiedFilterChain执行流程：**1、先执行Shiro自己的Filter链；2、再执行Servlet容器的Filter链（即原始的Filter）。**

而ProxiedFilterChain是通过FilterChainResolver根据配置文件中[urls]部分是否与请求的URL是否匹配解析得到的。 

```java
FilterChain getChain(ServletRequest request, ServletResponse response, FilterChain originalChain);  
```

即传入原始的chain得到一个代理的chain。

Shiro内部提供了一个路径匹配的FilterChainResolver实现：PathMatchingFilterChainResolver，其根据[urls]中配置的url模式（默认Ant风格）=拦截器链和请求的url是否匹配来解析得到配置的拦截器链的；而PathMatchingFilterChainResolver内部通过FilterChainManager维护着拦截器链，比如DefaultFilterChainManager实现维护着url模式与拦截器链的关系。因此我们可以通过FilterChainManager进行动态动态增加url模式与拦截器链的关系。

DefaultFilterChainManager会默认添加org.apache.shiro.web.filter.mgt.DefaultFilter中声明的拦截器：  

```java
public enum DefaultFilter {  
    anon(AnonymousFilter.class),  
    authc(FormAuthenticationFilter.class),  
    authcBasic(BasicHttpAuthenticationFilter.class),  
    logout(LogoutFilter.class),  
    noSessionCreation(NoSessionCreationFilter.class),  
    perms(PermissionsAuthorizationFilter.class),  
    port(PortFilter.class),  
    rest(HttpMethodPermissionFilter.class),  
    roles(RolesAuthorizationFilter.class),  
    ssl(SslFilter.class),  
    user(UserFilter.class);  
}  
```

下一节会介绍这些拦截器的作用。

 

如果要注册自定义拦截器，IniSecurityManagerFactory/WebIniSecurityManagerFactory在启动时会自动扫描ini配置文件中的[filters]/[main]部分并注册这些拦截器到DefaultFilterChainManager；且创建相应的url模式与其拦截器关系链。如果使用Spring后续章节会介绍如果注册自定义拦截器。



如果想自定义FilterChainResolver，可以通过实现WebEnvironment接口完成：  



```java
public class MyIniWebEnvironment extends IniWebEnvironment {  
    @Override  
    protected FilterChainResolver createFilterChainResolver() {  
        //在此处扩展自己的FilterChainResolver  
        return super.createFilterChainResolver();  
    }  
}   
```

FilterChain之间的关系。如果想动态实现url-拦截器的注册，就可以通过实现此处的FilterChainResolver来完成，比如：  

```java
//1、创建FilterChainResolver  
PathMatchingFilterChainResolver filterChainResolver =  
        new PathMatchingFilterChainResolver();  
//2、创建FilterChainManager  
DefaultFilterChainManager filterChainManager = new DefaultFilterChainManager();  
//3、注册Filter  
for(DefaultFilter filter : DefaultFilter.values()) {  
    filterChainManager.addFilter(  
        filter.name(), (Filter) ClassUtils.newInstance(filter.getFilterClass()));  
}  
//4、注册URL-Filter的映射关系  
filterChainManager.addToChain("/login.jsp", "authc");  
filterChainManager.addToChain("/unauthorized.jsp", "anon");  
filterChainManager.addToChain("/**", "authc");  
filterChainManager.addToChain("/**", "roles", "admin");  
  
//5、设置Filter的属性  
FormAuthenticationFilter authcFilter =  
         (FormAuthenticationFilter)filterChainManager.getFilter("authc");  
authcFilter.setLoginUrl("/login.jsp");  
RolesAuthorizationFilter rolesFilter =  
          (RolesAuthorizationFilter)filterChainManager.getFilter("roles");  
rolesFilter.setUnauthorizedUrl("/unauthorized.jsp");  
  
filterChainResolver.setFilterChainManager(filterChainManager);  
return filterChainResolver;   
```

此处自己去实现注册filter，及url模式与filter之间的映射关系。可以通过定制FilterChainResolver或FilterChainManager来完成诸如动态URL匹配的实现。

 

然后再web.xml中进行如下配置Environment：    



```xml
<context-param>  
<param-name>shiroEnvironmentClass</param-name> <param-value>com.github.zhangkaitao.shiro.chapter8.web.env.MyIniWebEnvironment</param-value>  
</context-param>   
```

####  自定义拦截器

通过自定义自己的拦截器可以扩展一些功能，诸如动态url-角色/权限访问控制的实现、根据Subject身份信息获取用户信息绑定到Request（即设置通用数据）、验证码验证、在线用户信息的保存等等，因为其本质就是一个Filter；所以Filter能做的它就能做。

 **1、扩展OncePerRequestFilter**

 OncePerRequestFilter保证一次请求只调用一次doFilterInternal，即如内部的forward不会再多执行一次doFilterInternal：   

```java
public class MyOncePerRequestFilter extends OncePerRequestFilter {  
    @Override  
    protected void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {  
        System.out.println("=========once per request filter");  
        chain.doFilter(request, response);  
    }  
}   
```



















