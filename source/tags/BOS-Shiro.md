---
title: BOS-Shiro
date: 2015-10-20 06:49:50
tags: shiro
---
### shiro

#### 权限表设计原理说明
首先，当用户访问没有地址时。根据用户请求资源路径-->查询数据库-->权限表（有无该访问的路径，如果有...doFilter放行）

要根据登录用户，拿到用户的id-->多表查询-->functions权限表 	 如果有这个路径就允许访问

分为用户表、角色表、权限表

用户和角色表是多对多的关系。

在User对象中有

```bash
private Set<Role> roles = new HashSet<Role>(0);
```

之后，给用户添加权限就相当于往用户的这个集合中添加Role。操作集合的增删改查就相当于操作中间表的增删改差

但是有一个前提，在修改的时候User必须是一个持久态。底层原理还是快照！！

#### shiro和spring整合
1. 将maven依赖导入到domain层
```bash
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-all</artifactId>
			<version>${shiro.version}</version>
		</dependency>
```
2. 在web.xml中添加shiro  filter
```bash
    <!-- 权限控制 Filter  放到 struts2前端控制器前面 -->
    <!-- shiro security filter -->
    <filter>
        <!-- 这里的 filter-name 要和 spring 的 applicationContext-shiro.xml 里的 org.apache.shiro.spring.web.ShiroFilterFactoryBean
            的 bean name 相同 -->
        <filter-name>shiroSecurityFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>shiroSecurityFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

这个Filter是spring提供，DelegationFilterProxy是代理Filter（会自动找 和"<"filter-name>"同名的bean>对象）
3. 在domain中创建applicationContext-shiro.xml
```bash
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
">
    <!--shiro认证和权限配置-->
    <!-- shiro 权限控制 -->
    <bean id="shiroSecurityFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!-- shiro 的核心安全接口 -->
        <property name="securityManager" ref="securityManager" />
        <!-- 要求登录时的链接 -->
        <property name="loginUrl" value="/login.jsp" />
        <!-- 登陆成功后要跳转的连接 -->
        <property name="successUrl" value="/index.jsp" />
        <!-- 权限不足，跳转路径  -->
        <property name="unauthorizedUrl" value="/unauthorized.jsp" />
        <!-- shiro 连接约束配置 -->
        <!-- URL控制规则  路径=规则 -->
        <property name="filterChainDefinitions">
            <value>
                /css/** = anon
                /demo/** = anon
                /images/** = anon
                /js/** = anon
                /json/** = anon
                /login.jsp** = anon
                /index.jsp** = authc
                /validatecode.jsp** = anon
                /user/userAction_login** = anon
                /user/userAction_validCheckCode** = anon
                /** = authc
                <!--//  除了上述配置 其他资源必须需要身份认证 (登陆)-->
            </value>
        </property>
    </bean>

    <!-- 安全管理器 -->
    <bean id="securityManager"
          class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <!-- 在安全管理器，应该注入 Realm 连接安全数据  -->
    </bean>

</beans>
```
到这里struts.xml的登录拦截就没用了。
而且这里按照之前的登录是登录不进去的，因为 Subject 和 Realm 都还没用创建。

#### shiro_Subject编写说明
由于使用了shiro所以原先的过滤器就没用了。
```bash
if (input_checkcode.equalsIgnoreCase(session_code)) {
				/*// 验证码一样，需要门面类去调dao
				User existUser = serviceFacade.getUserService().login(model.getUsername(), model.getPassword());
				if (existUser == null) {
					// 数据库没查到
					this.addActionError(this.getText("login.usernameOrPassword.error"));
					return "login_error";
				} else {
					// 将用户信息保存到session中
					setSessionAttribute("existUser", existUser);
					return "login_ok";
				}*/
				//shiro  Subject 接收表单数据 封装  令牌对象中
				//1、获取subject
				Subject subject=SecurityUtils.getSubject();
```
Subject 是一个接口，所以不能实例化对象。要通过SecurityUtils.getSubject();来获取。
```bash
Subject subject=SecurityUtils.getSubject();
```
Subject实体类只封装令牌不做数据访问，因此需要一个令牌实体。
```bash
UsernamePasswordToken usernamePasswordToken=new UsernamePasswordToken(model.getUsername(),model.getPassword());
```
登录
```bash
//2、登录  subject只封装令牌不做数据访问
	subject.login(usernamePasswordToken);  //login方法抛出异常  表示 登录身份失败 该方法没有异常认证成功
```
通过是否抛出异常来判断是否登录成功。为subject的login方法添加try/catch
#### shiro_realm认证编码说明
Shiro是独立于web三层框架的。

首先，需要编写一个 Realm 然后注入到 securityManager 中
```bash
public class BosRealm extends AuthorizingRealm {
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //授权管理
        System.out.println("---授权管理---");
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //认证  realm  调用底层数据  和数据库比对  将bosrealm依赖注入安全管理器 SecurityManager
        System.out.println("---认证---");
        return null;
    }
```
注入到 securityManager
```bash
    <bean id="bosRealm" class="com.zero.bos.realm.BosRealm"></bean>


    <!-- 安全管理器 -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <!-- 在安全管理器，应该注入 Realm 连接安全数据  -->
        <property name="realm" value="bosRealm"></property>
    </bean>
```
doGetAuthorizationInfo :用于授权管理

AuthenticationInfo ：登录认证

realm的底层是 调用底层数据和数据库进行比对

如果 doGetAuthenticationInfo 方法的返回值是NULL  那么就表示身份认证失败了  如果不为NULL 返回 AuthenticationInfo 表示认证成功

参数 AuthenticationToken 令牌就是  Subject.login（令牌） 拿到令牌就有了表单账户密码

拿到令牌之后，调用业务层的dao查询数据库  返回User对象，如果查询数据库对象是存在的就进行认证流程

认证流程注重密码：（1）账户是要保证唯一性的 （2）密码是加密的 只能重置不能直接拿到密码 

令牌里的密码  和 查询用户得到的密码 进行比对


#### shiro_realm认证编码实现


```bash
    public SimpleAuthenticationInfo(Object principal, Object credentials, String realmName) {
        this.principals = new SimplePrincipalCollection(principal, realmName);
        this.credentials = credentials;
    }
```
BosRealm代码
```bash
UsernamePasswordToken userToken=(UsernamePasswordToken) token; //拿到密码和用户名

        String username=userToken.getUsername(); //通过账户拿密码
        //2、查询数据库
        User existUser= serviceFacade.getUserService().findUserByUsername(username);
        //进行判断
        if (existUser==null){
            return null;
        }else {
            //比对密码 AuthenticationInfo 对象 有三个参数 1、当前登录的用户信息  为后续获取用户信息准备
            // 仅仅是为Subject提供信息不做比对 令牌才是比对
            //参数2、数据库里面的真实密码   底层自动和令牌的密码比对 如果错误info是空值
            //参数3：当前realm 在容器中 bean 的 id值  一般用方法拿而不是固定的名称
            SimpleAuthenticationInfo info=new SimpleAuthenticationInfo(existUser,existUser.getPassword(),super.getName());
            return info;
        }
```
过程
1. 首先通过传递过来的token拿到账户
2. 通过账户去查询是否存在有这个账户的对象
3. 拿到对象后进行对比密码
SimpleAuthenticationInfo的三个参数：（1）当前登录的用户信息,仅仅为Subject提供数据，不做比对  （2）数据库里面的真实密码  （3）当前realm在容器中 bean 的id值，一般用方法获取而不是固定的"BosRealm"

认证只走一次，Realm这个方法。

如果密码错误后台会报错：

```bash
org.apache.shiro.authc.UnknownAccountException    //账户错误
org.apache.shiro.authc.IncorrectCredentialsException  //密码错误
```
提升用户体验度
在UserAction中添加代码
```bash
				UsernamePasswordToken token=new UsernamePasswordToken(model.getUsername(),model.getPassword());

				//2、登录  subject只封装令牌不做数据访问
				try {
					subject.login(token);  //login方法抛出异常  表示 登录身份失败 该方法没有异常认证成功
					return "login_ok";
				} catch (UnknownAccountException e) {
					e.printStackTrace();
					this.addActionError(this.getText("login.username.error"));
					return "login_error";
				}catch (IncorrectCredentialsException e) {
					e.printStackTrace();
					this.addActionError(this.getText("login.password.error"));
					return "login_error";
				}
```
国际化
```bash
login.username.error=账户不存在
login.password.error=密码错误
```

#### shiro认证页面显示客户信息以及退出系统

因为这个时候不再使用底层的session了。所以不显示本地的IP

```javascript
<div id="sessionInfoDiv"
   style="position: absolute;right: 5px;top:10px;">
   [<strong>超级管理员</strong>]，欢迎你${sessionScope.existUser.username}!
   您使用[<strong>${pageContext.request.remoteAddr}</strong>]IP登录！
</div>
```

使用shiro标签库来获取

- 引入标签库

```javascript
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>
```

- 通过标签获取用户名

  ```javascript
  <div id="sessionInfoDiv"
     style="position: absolute;right: 5px;top:10px;">
     [<strong>超级管理员</strong>]，欢迎你<shiro:principal property="username"></shiro:principal>!
     您使用[<strong>${pageContext.request.remoteAddr}</strong>]IP登录！
  </div>
  ```



退出登录修改

**原来的代码**

```javascript
// 退出登录
function logoutFun() {
   $.messager
   .confirm('系统提示','您确定要退出本次登录吗?',function(isConfirm) {
      if (isConfirm) {
         location.href = '${pageContext.request.contextPath }/login.jsp';
      }
   });
```

**改为**

```javascript
// 退出登录
function logoutFun() {
   $.messager
   .confirm('系统提示','您确定要退出本次登录吗?',function(isConfirm) {
      if (isConfirm) {
         location.href = '${pageContext.request.contextPath }/user/userAction_logout';
      }
   });
}
```

#### shiro授权临时数据录入

1. 首先模拟一下权限不足的场景

```java
/page_base_region** = roles["base"]  <!--需要有base这个角色才能访问这个路径 -->
```

上面这行代码表示  需要有base这个角色才可以访问这个路径

在 struts.xml中

```xml
<!-- 需要进行权限控制的页面访问 -->
<action name="page_*_*">
   <result type="dispatcher">/WEB-INF/pages/{1}/{2}.jsp</result>
</action>
```

那么这个时候，region页面放就访问不了了。

![](https://github.com/whyathere/tuchuang/blob/master/Shiro/unauthorized.jpg?raw=true)

当当前用户没有角色或者权限的时候就会向这里跳转

```xml
<!-- 权限不足，跳转路径  -->
<property name="unauthorizedUrl" value="/unauthorized.jsp" />
```

思路：根据用户，查找到用户的ID，然后通过ID去查用户角色表

```java
SimpleAuthorizationInfo //实现类  提供 addRole（String） 角色 addStringPemission（String）权限
```

根据登录用户，查询 该用户具有的角色 拿到角色表设计code 值 添加到 该对象集合中/获取权限code 添加该对象 对象集合中

addRole（String） 里的String 和xml文件中 roles["base"]   的base进行比较

```java
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
    //AuthorizationInfo 授权管理 授权实现：根据当前登录用户  获取用户id  查询该用户在数据库表中  具有哪些角色 或者权限
    System.out.println("---授权---");
    //1号 jack 1 角色 code：base 可以访问  region 资源
    //2 号 rose 2角色 code：qupai 不可以访问 region 资源
    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
    Subject subject = SecurityUtils.getSubject();
    User existUser = (User)subject.getPrincipal();
    //系统  超级管理员 和登录用户 账户是唯一的
    if ("tom".equals(existUser.getUsername())){
        //将系统中所有的角色关键字 code 和所有的权限 code 都添加 Info
        List<Role> roles=facadeService.getRoleService().findAllRoles();
        for (Role role:roles){
            info.addRole(role.getCode()); //具有所有角色的关键字
        }
        List<Function> functions=facadeService.getFunctionService().findAllFunctions();
        for (Function function:functions){
            info.addStringPermission(function.getCode());
        }
    }else {
        //登录用户具有权限  根据当前登录用户 id  查询 该用户具有角色 以及 每个对应角色 对应权限

    }
    return info;
}
```

创建RoleService、FunctionService、Impl以及Dao

```java
subject.getPrincipal()  //获取对象
```

判断用户名是否是"tom".如果是的话就把所有的权限和用户关键字给到tom。

返回info

#### shiro授权之非超级管理员权限realm实现

```java
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
    //AuthorizationInfo 授权管理 授权实现：根据当前登录用户  获取用户id  查询该用户在数据库表中  具有哪些角色 或者权限
    System.out.println("---授权---");
    //1号 jack 1 角色 code：base 可以访问  region 资源
    //2 号 rose 2角色 code：qupai 不可以访问 region 资源
    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
    Subject subject = SecurityUtils.getSubject();
    User existUser = (User) subject.getPrincipal();
    //系统  超级管理员 和登录用户 账户是唯一的
    if ("tom".equals(existUser.getUsername())) {
        //将系统中所有的角色关键字 code 和所有的权限 code 都添加 Info
        List<Role> roles = facadeService.getRoleService().findAllRoles();
        for (Role role : roles) {
            info.addRole(role.getCode()); //具有所有角色的关键字
        }
        List<Function> functions = facadeService.getFunctionService().findAllFunctions();
        for (Function function : functions) {
            info.addStringPermission(function.getCode());
        }
    } else {
        //登录用户具有权限  根据当前登录用户 id  查询 该用户具有角色 以及 每个对应角色 对应权限
        List<Role> roles = facadeService.getRoleService().findAllRolesByUserId(existUser.getId());
        for (Role role : roles) {
            info.addRole(role.getCode());
            Set<Function> functions = role.getFunctions();  //每一个角色对应多个权限
            for (Function function : functions) {
                info.addStringPermission(function.getCode());
            }
        }
    }
    return info;
}
```

```java
//角色 role user_role 对象连接
public interface RoleDao extends JpaRepository<Role,String> {

    @Query("from Role r inner join r.users u where u.id = ?1")
    public List<Role> findAllRolesByUserId(String id);
```

Query语句说明：内连接，需要Role和 中间表（user_role）都存在，join r.users就连接上了。然后查找u.id 因为在user_role 有user也有role。

对于jpa来说，它做多变关系连接的时候，它有一个叫做迫切内连接和非迫切内连接。同样的也有迫切外连接和非迫切外连接。

**区别**：

- **非迫切内连接 ：** 封装的结果集是Object[]。查的 时候并不是封装到List<Role> 而是 List<Object[]>而Object数组内部有两个元素，一个是Role一个是Role的关联Function
- **迫切内连接 ：**查询的结果集  封装单一的对象里面，在对象的内部引入了关联的集合。Function

如果想封装到单一的Role中，就需要用迫切。就必须要用关键字fetch，非迫切就不需要fetch。

而我们的数据结果集是要封装到Role中的要加fetch

```java
@Query("from Role r inner join fetch r.users u where u.id = ?1")
public List<Role> findAllRolesByUserId(String id);
```

这样Jpa会强制把结果集封装到单一的对象里去。

如果不加fetch会放到一个Object集合中，第一个元素是Role第二个是Function集合

这里泛型是和from后面跟的对象一致的

这里就可以看出code关键字是base的可以访问“区域”页面。而没有这个关键字的不能访问。

#### shiro注解权限控制配置说明

目前想要访问region页面必须角色有这个的关键字"base"，现在jack 和tom 有，而rose没有

![](https://github.com/whyathere/tuchuang/blob/master/Shiro/%E8%A7%92%E8%89%B2%E6%88%AA%E5%9B%BE.jpg?raw=true)

rose是不能查询取派员的。

方法级别只需要添加一个注解就可以给到这个权限了。

```java
// 取派员分页查询
@Action(value = "staffAction_pageQuery")
@RequiresRoles(value = "base")  //访问这个资源就需要有base权限
public String pageQuery() {
   Map<String,Object> data=new HashMap<String,Object>();
   try {
      Page <Staff> pageData=serviceFacade.getStaffService().pageQuery(getPageRequest());
      setPageData(pageData); //提供给父类
      //dao  参数   Pageable pageable   自动完成分页查询   将分页结果数据   自动封装到Page<T>
      //从Page  对象获取总记录数  和   每页分页记录数List<Staff>
   } catch (Exception e) {
      e.printStackTrace();
      push(false);
   }
   return "pageQuery";  //走全局结果集
}
```

@RequiresRoles(value = "base")   //访问这个资源就需要有base权限

**注解方式原理：**

首先目前就算加上注解还是不能用的。

shiro 注解原理：注解信息是通过反射来读的

涉及到动态代理：代理类对目标类方法进行加强。  方法是：pageQuery。采用了Spring 的aop来实现的。

aop：面向接口代理

​	代理类 $proxy。。和 目标类  StaffAction 的关系：

​	兄弟关系    都属于接口实现类

代理类：面向目标类代理

代理类和目标类  是 父子关系

代理类是目标类  StaffAction 子类

默认向  StaffAction  接口做代理类  ，shiro注解的代理类是向接口做代理



shiro注解配置权限需要：

1. 开启注解配置aop
2. 目标方法上添加注解即可

```xml
<!--使用注解完成 权限管理  启用注解 需要配置3个bean-->
<!-- spring bean 对象后处理器  -->
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

<!-- 切面自动代理 bean生成注解代理类实现者 修改代理类生成策略 面向接口 默认面向接口代理 -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
      depends-on="lifecycleBeanPostProcessor"/>

<!-- 切面 -->
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
    <property name="securityManager" ref="securityManager"/>
</bean>
```



这个时候取派员页面，tom可以正常访问显示。jack报错。

```java
11:42:00,780 ERROR Dispatcher:38 - Exception occurred during processing request: com.sun.proxy.$Proxy58.pageQuery()

java.lang.NoSuchMethodException: com.sun.proxy.$Proxy58.pageQuery()

```

原因：默认是面向接口做代理，当前类有接口。但是当前类的接口没有pageQuery方法。

```xml
<!-- 切面自动代理 -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
      depends-on="lifecycleBeanPostProcessor"/>
```

这个bean负责生成代理类。

改为

```xml
<!-- 切面自动代理 -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
      depends-on="lifecycleBeanPostProcessor">
    <property name="proxyTargetClass" value="false"></property>
</bean>
```

这样就从面向接口变为面向类

这个时候还是木有啊！！

新的异常

![](https://github.com/whyathere/tuchuang/blob/master/Shiro/shiro%E5%BC%82%E5%B8%B8.jpg?raw=true)

这里是报的父类的错，这说明类型不一样。

父类获取子类参数化泛型  父类可以获取到  staff  可以进行实例化。

BaseAction无法获取代理类泛型！

默认是Object类型

原因：代理类没有泛型，代理类是动态生成的。泛型在编译的时候有效，运行的时候就没有了。

泛型的作用周期是编译期，运行的时候所有的泛型就都消失了。

解决方法，不要从代理类去找。去父类找

```java
// 目的 获取子类参数化类型
// 父类构造方法中 使用 参数化泛型+反射 获取子类具体泛型对应类型 newInstance 创建实例
public BaseAction() {
    // 对model进行实例化， 通过子类 类声明的泛型
    Type superclass = this.getClass().getGenericSuperclass();
    if (!(superclass instanceof ParameterizedType)){
        //没有代理类
      superclass =  this.getClass().getSuperclass().getGenericSuperclass();
    }
    // 转化为参数化类型
    ParameterizedType parameterizedType = (ParameterizedType) superclass;
    // 获取一个泛型参数
    Class<T> modelClass = (Class<T>) parameterizedType.getActualTypeArguments()[0];
    try {
        model = modelClass.newInstance();
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
}
```

这次报了空指针异常

java.lang.NullPointerException

原因：在FacadeService中

```java
@Autowired
// 注入 业务层实例
private StaffService staffService;
```

这个注解没有将接口实例注入给接口

![](https://github.com/whyathere/tuchuang/blob/master/Shiro/%E5%8E%9F%E5%9B%A0.jpg?raw=true)

就是代理类没有继承@Autowried注解

```xml
<!--更改默认配置文件信息-->
<constant name="struts.objectFactory.spring.autoWire.alwaysRespect" value="true"/>
```

在struts.xml中添加这个

让代理类可以继承AutoWired注解





































































































































































