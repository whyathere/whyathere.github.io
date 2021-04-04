---
title: Mybatis
date: 2018-09-10 18:27:21
tags: Mybatis
categories: 框架
---

### 全局参数配置

#### 批量起别名

```xml
<typeAliases>
    <package name="com.zero.dao"/>
</typeAliases>
```

```xml
<settings>
    <!--默认是OTHER这里改为NULL-->
    <setting name="jdbcTypeForNull" value="NULL"/>
    <!--开启驼峰命名-->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

### Mybatis参数

#### 主要参数的意义

property:对应到javaBean中的值

column:指定数据库中那一列的值

javaType:列值对应的java类型

discriminator:标签鉴别器

```xml
<!--获取自增主键的值
    useGeneratedKeys 使用自增主键获取主键值策略
    keyProperty : 指定对应的主键属性，也就是mybatis获取到主键值以后，将这个值封装给javaBean的那个属性
        -->
<insert id="addEmployee" useGeneratedKeys="true" keyProperty="id">
    insert into tb_user (name,address)
    values
    (#{name},#{address})
</insert>
```

#### @Param

多个参数会被封装成一个map，#{}就是从map中获取指定的key的值。

key是param

value是传入的值

```xml
    <select id="getBlogByIdAndName" resultType="com.zero.dao.Blog">
        select * from tb_user where id = #{param1} and name = #{param2}
    </select>
```

多个参数的时候可以命名参数：明确指定封装参数时map的key。@Param("")

```java
public Blog getBlogByIdAndName(@Param("id") Integer id, @Param("name") String name);
```

key：使用@Param注解指定的值

value:参数值

#{指定的key}取出对应的参数值

```xml
    <select id="getBlogByIdAndName" resultType="com.zero.dao.Blog">
        select * from tb_user where id = #{id} and name = #{name}
    </select>d
```



#### 以Map为参数集合

如果多个参数正好是我们业务逻辑的数据模型，我们就可以直接传入pojo

#{属性名}：取出传入pojo的属性值

如果多个参数不是业务模型中的数据，没有对应的pojo，不经常使用为了方便，我们可以传入map

#{key}:取出map中对应的值

```java
 public Blog getBlogByMap(Map<String,String> map);
```

```xml
    <select id="getBlogByMap" resultType="com.zero.dao.Blog">
        select *  from tb_user where id = #{id} and name = #{name}
    </select>
```

```java
            Map<String,String> map = new HashMap<>();
            map.put("id","4");
            map.put("name","zhangsan");
            Blog blog = mapper.getBlogByMap(map);
```

#### 编写专门的TO

如果多个参数不是业务模型中的数据，但是经常要使用，推荐来编写一个TO(Transfer Object)数据传入对象

Page{

​	int index;

​	int size;

}



#### 传参处理

```java
public Blog getBlog(@Param("id") Integer id,  String name);
```

取值：id==>#{id/param1}   name ==> #{param2}

```java
public Blog getBlog(Integer id,  @Param("e")Blog blog);
```

取值：id ==> #{param1}   name ==>  #{param2.name/e.name}

特别注意：如果是Collection (List、Set) 类型或者数组，也会特殊处理。也是把传入的list或者数组封装在map中，

key：Collection(collection),如果是List还可以使用这个key(list)

数组(array)



```java
public Blog getBlogById(List<Integer> ids);
```

取值：取出第一个id的值: #{list[0]}   不能写ids或者param1

### #和$取值

#{}和${}都可以取map或者pojo的值

   ```sql
select *from tb_blog where id = 2 and name = ?
   ```

区别：

​	#{}：是以预编译的形式，将参数设置到sql语句中；？PreparedStatement  防止sql注入

​	${}：取出的值直接拼装在sql语句中；会有安全问题，无法防止sql注入

大多情况下取参数的值都应该使用#{}；

比如分表、排序：按照年份拆分

​	select * from 2016(2017)_year

年份不确定，原生jdbc不支持占位符的地方我们可以使用${}进行取值

```sql
select * from ${year}_salary where xxx; 
```

#{}只能取出参数中的值

#{}：更丰富的用法：

规定参数的一些规则：

javaType、jdbcType、mode(存储过程)、numericScale、

resultMap、typeHandler、jdbcTypeName、expression

jdbcType：通常要在某种特定的条件下设置；

​	在我们的数据为null的时候，有些数据库可能不能识别mybatis对null的默认处理。比如oracle：为null时会报错，这个时候要处理这个报错告知mybatis。

```java
Blog blog = new Blog(null,"lisi","null");
mapper.addBlog(blog);
```

​	如果是oracle会报错：无效的类型。不能区分出null的类型。JdbcType OTHER：无效的类型

而mysql的没有问题的。因为mybatis对所有的null都映射的是JDBC OTHER类型。

当jdbcType为null时，默认为OTHER。这个OTHER  oracle不认识。由于全局配置中，jdbcTypeForNull = OTHER

1、#{enmail,jdbcType=OTHER} 只会影响当前sql语句的jdbcType

2、直接改全局配置，将全部的jdbcTypeForNull改为null

```xml
<settings>
    <!--默认是OTHER这里改为NULL-->
    <setting name="jdbcTypeForNull" value="NULL"/>
</settings>
```

这个NULL，oracle和mysql都可识别。

### 查询

select元素来定义查询操作

id:唯一标识符

-用来引用这条语句，需要和接口的方法名一致

parameterType：参数类型

-可以不传，Mybatis会根据TypeHandler自动推断

resultType：返回值类型

-别名或者全类名，如果返回的是集合，定义集合中元素的类型。不能和resultMap同时使用。

```java
public List<Blog> getBlogByNameLike(String name);
```

```xml
    <!--resultType:如果返回值类型是一个集合，要写集合中元素的类型-->
    <select id="getBlogByNameLike" resultType="com.zero.dao.Blog">
      select * from tb_user where name like #{name}
    </select>
```

```java
            BlogMapper mapper = session.getMapper(BlogMapper.class);
//            Blog blog = new Blog(null,"zhangsan","Beijing");
            List<Blog> like = mapper.getBlogByNameLike("%z%");
            for (Blog blog : like){
                System.out.println(blog);
            }
```

resultType为com.zero.dao.Blog时，jdbc会将数据封装为Blog然后返回。

#### select记录封装map

```java
    @MapKey("name")
    //告诉mybatis封装这个map的时候使用哪个属性作为map的key
    public Map<String,Blog> getBlogByNameLikeReturnMap(String name);
```

@MapKey是指定封装这个map的时候哪个属性作为封装Map的key

```xml
    <!--Map<String,Blog>   -->
    <select id="getBlogByNameLikeReturnMap" resultType="com.zero.dao.Blog">
        select * from tb_user where name like #{name}
    </select>
```

```java
Map<String, Blog> map = mapper.getBlogByNameLikeReturnMap("%w%");
```

#### 自定义结果映射规则 resultMap

1. 如果查出的列明和javaBean的属性名不一样，需要写别名，或者如果符合驼峰命名法则开启驼峰命名

   - autoMappingBehavior默认是PARTIAL，开启自动映射的功能。唯一的要求是列名和javaBean属性名一致

   - 如果autoMappingBehavior设置为null则会取消自动映射

   - 数据库字段命名规范，POJO属性符合驼峰命名法，如A_COLUMN->aColumn，我们可以开启自动驼峰命名规则映射功能

   - ```xml
     <!--开启驼峰命名-->
     <setting name="mapUnderscoreToCamelCase" value="true"/>
     ```


此外resultMap还可以指定自定义结果集映射规则

如何定义规则：在该mapper标签中最上面自定义某个javaBean的封装规则

如下：

```xml
<mapper namespace="com.zero.dao.BlogMapperPlus">
    <!--自定义某个javaBean的封装规则
        type：自定义规则的Java类型
        id：唯一id方便引用
    -->
    <resultMap id="MyBlog" type="com.zero.dao.Blog">
        <!--指定主键的封装规则
        id：定义主键会底层有优化
        column:指定那一列
        property：指定对应的javaBean属性
        -->
        <id column="id" property="id"/>
        <!--定义普通列封装规则-->
        <result column="name" property="name"/>
        <!--其他不指定的列会自动封装，但是只要写resultMap就要把全部的映射规则都写上-->
        <result column="address" property="address"/>
    </resultMap>

    <select id="getBlogById" resultMap="MyBlog">
        select * from tb_user where id = #{id};
    </select>
</mapper>	
```

#### resultMap关联查询

如果Blog中包含部门对象

现在的要求是在查出这个对象的同时查出其所在的部门

Department

```sql
CREATE TABLE tbl_dept(
id INT(11) PRIMARY KEY AUTO_INCREMENT,
dept_name VARCHAR(255)
)

ALTER TABLE tb_user ADD COLUMN d_id INT(11);
```

创建tbl_dept表并为tb_user表添加一列d_id

现在为tb_user的d_id属性添加外键

```sql
ALTER TABLE tb_user add CONSTRAINT fk_user_dept
FOREIGN KEY(d_id) REFERENCES tbl_dept(id);
```

外键名称为fk_user_dept，将d_id的外键设置为tbl_dept的id属性。

此时想要查询出所有信息有两种方法：

1、外键关联

```xml
<!--联合查询：级联属性封装结果-->
<resultMap id="MyResult" type="com.zero.dao.Blog">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="address" property="address"/>
     <!--可以指定联合的javaBean对象
        property="dept" 指定那个属性是联合对象
     -->
    <result column="did" property="dept.id"/>
    <result column="dept_name" property="dept.departmentName" />
</resultMap>

    <select id="getBlogAndDeptById" resultMap="MyResult">
      select e.id id,e.name name ,e.address address,d.id did,d.dept_name dept_name from tb_user e,tbl_dept d
      where e.d_id=d.id and e.id = #{id};
    </select>
```

方法2：association标签

```xml
<!--联合查询：级联属性封装结果-->
<resultMap id="MyResult" type="com.zero.dao.Blog">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="address" property="address"/>
     <!--可以指定联合的javaBean对象
        property="dept" 指定那个属性是联合对象
     -->
    <association property="dept"  javaType="com.zero.dao.Department">
    <id column="did" property="id"/>
    <result column="dept_name" property="departmentName"/>
    </association>
</resultMap>
```

```java
@Test
public void test01() throws IOException {
    String resource = "mybatis/mybatis-config.xml";
    InputStream is = Resources.getResourceAsStream(resource);
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
    SqlSession session = sessionFactory.openSession(true);
    try {
        BlogMapperPlus mapper = session.getMapper(BlogMapperPlus.class);
        Blog blog = mapper.getBlogAndDeptById(5);
        System.out.println(blog);
        System.out.println(blog.getDept());
    } finally {
        session.close();
    }
}
```

注意事项：这里测试的时候有报错信息：No constructor found in

Bean里面有一个全参数的构造函数，但是字段类型不匹配。

此时需要检查两个相互关联的javaBean是否有构造方法，如果设置了有参的构造方法，那么还需要有默认的无参构造。两个javaBean都需要。

方法三：association分布查询

先查询user然后拿到d_id之后再去部门里查询部门信息，这就可以写两个sql。一个比较复杂

```java
public interface DepartmentMapper {
    public Department getDeptById(Integer id);
}
```

```xml
BlogMapperPlus.xml
    <!--使用association进行分步查询
        1、先按照员工id查询员工信息
        2、根据查询员工信息中的d_id值去部门表查出部门信息
        3、部门设置员工中
    -->
    <resultMap id="MyResult1" type="com.zero.bean.Blog">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="address" property="address"/>
        <!--
        association定义关联的对象的封装规则.dept的值是调用查询方法查出来的.
        select:表明当前的属性是调动select指定的方法查出的结果
        column:指定将哪一列的值传递给这个方法
        流程：使用select指定的方法(传入column指定的这列参数的值)查出对象，并封装给property指定的属性
        --> 
        <association property="dept" select="com.zero.dao.DepartmentMapper.getDeptById" column="d_id">

        </association>
    </resultMap>
    
    <select id="getBlogByIdStep" resultMap="MyResult1">
        select * from tb_user where id=#{id}
    </select>
```

```java
BlogMapperPlus mapper = session.getMapper(BlogMapperPlus.class);
Blog dept = mapper.getBlogByIdStep(5);
```

##### 分布查询&延迟加载

每次查询user的时候，都将一起查询出来。部门信息在我们使用的时候再去查询；

需要在分段查询的基础上再加两个配置

```xml
        <!--开启延迟加载 显示的指定每个我们需要更改的配置的值，即使他是默认的，防止更新带来的问题-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!--在需要任何属性的时候，属性都会被加载，否则属性会在被需要的时候加载-->
        <setting name="aggressiveLazyLoading" value="false"/>
```

```java
BlogMapperPlus mapper = session.getMapper(BlogMapperPlus.class);
Blog dept = mapper.getBlogByIdStep(5);
System.out.println(dept.getName());
```

##### collection定义关联集合封装规则

场景二：查询部门的时候将部门对应的所有员工信息也查询出来。

<collection>定义关联集合类型的属性的封装规则

```xml
    <!--嵌套结果集方式，使用collection标签定义关联的集合类型的属性封装规则-->
    <resultMap id="MyDept" type="com.zero.bean.Department">
        <id column="id" property="id"/>
        <result column="dept_name" property="departmentName"/>
        <!--collection定义关联集合类型的属性的封装规则
            这里指定blogs是一个集合
            ofType:指定集合里面元素的类型
        -->
        <collection property="blogs" ofType="com.zero.bean.Blog">
            <!--定义集合中元素的封装规则-->
            <id column="eid" property="id"/>
            <result column="name" property="name"/>
            <result column="address" property="address"/>
        </collection>
    </resultMap>

    <!--getDeptByIdPlus-->
    <select id="getDeptByIdPlus" resultMap="MyDept">
        select d.id did,d.dept_name dept_name,e.id eid,e.name name,e.address address
        from tbl_dept d
        left join tb_user e
        on d.id=e.d_id
        where d.id=#{id}
    </select>
```

此时查询结果时出现两个问题：

- 不显示department的id，id为null
- 该部门有多个员工时提示报错。

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/Mybatis/found%EF%BC%9A2.png?raw=true)

上面两个问题都是由于一个原因：

在DepartmentMapper.xml中

```sql
select d.id did   查询出来的是以did命名的
```

而resultMap

```xml
<id column="id" property="id"/>
```

column依然是id，这里改为did即可。

##### collection分布查询&延迟加载

分不查询可以先查dept中id为1的对象

然后再查employee中did也为1的对象

操作如下

```java
public List<Employee> getEmpsByDeptId(Integer id);
```

```xml
<select id="getEmpsByDeptId" resultType="com.zero.bean.Employee" >
    select * from tb_user where d_id=#{id}
</select>
```

DepartmentMapper

```java
public Department getDeptByIdStep(Integer id);
```

```xml
<resultMap id="MyDeptStep" type="com.zero.bean.Department">
    <id column="id" property="id"/>
    <result column="dept_name" property="departmentName"/>
    <collection property="employees" select="com.zero.dao.EmployeeMapperPlus.getEmpsByDeptId" column="id"/>
</resultMap>
<!--getDeptByIdStep-->
<select id="getDeptByIdStep" resultMap="MyDeptStep">
    select id,dept_name dept_name from tbl_dept where id=#{id}
</select>
```

column是指定要把那一列的值传给这个方法

如果要将多列的值传入要怎么操作呢？

```xml
<resultMap id="MyDeptStep" type="com.zero.bean.Department">
    <id column="id" property="id"/>
    <result column="dept_name" property="departmentName"/>
    <collection property="employees" select="com.zero.dao.EmployeeMapperPlus.getEmpsByDeptId" column="{deptId=id}"  />
</resultMap>
<!--getDeptByIdStep-->
<select id="getDeptByIdStep" resultMap="MyDeptStep">
    select id,dept_name dept_name from tbl_dept where id=#{id}
</select>

<!--扩展 collection要传递多列的值
    column="{key1=column1,key2=column2}"
    fetchType="lazy"  ：表示使用延迟加载
    lazy：延迟
    eager：立即
-->
```

##### discriminator标签鉴别器

mybatis可以通过鉴别器识别某个列的值并作出相应的操作。

例如：如果是女生就把该女生的部门信息查询出来

​	   如果是男生就不查询部门信息，但是要把name赋值给email

```xml
<resultMap id="MyEmpDis" type="com.zero.bean.Employee">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="address" property="address"/>
    <!--mybatis可以使用鉴别器判某列的值，然后根据某列的值改变封装行为
        封装Employee:
            如果查出的是女生 0 就把部门信息查询出来，否则不查。
            如果是男生 1，把name这一列的值赋值给email
        column:指定要判定的列名
        javaType:列值对应的java类型

    -->
    <discriminator javaType="string" column="gender">
        <!--女生 resultType:列值对应的java类型.也可以是resultMap但是不能缺少-->
        <case value="0" resultType="com.zero.bean.Employee">
            <association property="dept" select="com.zero.dao.DepartmentMapper.getDeptById" column="d_id">
            </association>
        </case>
        <!--男生把name赋值给email-->
        <case value="1" resultType="com.zero.bean.Employee">
            <id column="id" property="id"/>
            <result column="name" property="name"/>
            <result column="name" property="email"/>
            <result column="gender" property="gender"/>
        </case>
    </discriminator>
</resultMap>
```

### DynamicSQL

- 动态SQL是Mybatis强大特性之一。极大的简化我们拼装SQL的操作。
- 动态SQL元素和使用JSTL或其他类似基于XML的文本处理器相似。
- MyBatis采用功能强大的基于OGNL的表达式来简化操作。
  - if
  - choose(when , otherwise)
  - trim(where,set)
  - foreach

#### if判断

```xml
 <!--查询员工，emp对象携带了那个字段就通过这个字段来查询或者叠加查询-->
    <select id="getEmpsByconditionIf" resultType="com.zero.bean.Employee">
        select * from tb_user
        where
        <if test="id!=null">
            id=#{id}
        </if>
        <!-- test:判断表达式（OGNL）  c:if test
             从参数中取值进行判断
             遇见特殊符号应该去写转义字符比如
              " :&quot;   或者 ''
              && ： &amp;&amp;
         -->
        <if test="name !=null and name !=''">
            and name like #{name}
        </if>
        <!--<if test="address !=null and address !='' ">-->
        <if test="address !=null &amp;&amp; address !=&quot;&quot; ">
            and address =#{address}
        </if>
        <if test="email !=null and email.trim()!=&quot;&quot;">
            and email=#{email}
        </if>
        <!--ognl会进行字符串与数字的转换判断 "0"==0  字符串0和数字0是一样的 -->
        <if test="gender==0 or gender==1">
            and gender=#{gender}
        </if>
    </select>
```

这里传入的参数是Employee，OGNL会自动识别里面的值

#### where查询条件

上面的如果没有id，那么sql语句就会直接变成select * from tb_user  where and address = ?

明显sql语句是错误的。

如果查询的时候如果某些条件没带sql拼装就会有问题

解决方法：

1. ```sql
   where  1==1
   ```

2. mybatis使用where标签来将所有的查询条件包括在内

将所有if放到<where></where>标签内即可。这样，mybatis会将where标签中多出来的and或者or去掉。但是有时候where也是不好使的。例如：

将and放在语句的后面。  email=#{email}

where就只会去掉第一个多出来的and或者or

```sql
select * from tb_user WHERE address =? and 
```

**所以在使用where标签的时候，and要写在第一位**

#### sql_trim自定义字符串截取

就像上面的where标签所描述的那样，如果把and或者or放在了sql的后面，那么只能识别第一个and或者or其他的就会报错。

```xml
<trim prefix="" prefixOverrides="" suffix="" suffixOverrides=""></trim>
```

trim有四个属性

- prefix：给拼串后的整个字符串加一个前缀
- prefixOverrides:前缀覆盖：去掉整个字符串前面多余的字符
- suffix：给拼串后的整个字符串加一个后缀
- suffixOverrides：去掉整个字符串后面多余的字符

```xml
    <select id="getEmpsByonditionTrim"  resultType="com.zero.bean.Employee">
            select * from tb_user
            <!-- 自定义截取规则 -->
        <trim prefix="where" suffixOverrides="and">
        <if test="id!=null">
            id=#{id}
        </if>
        <!-- test:判断表达式（OGNL）  c:if test
             从参数中取值进行判断
             遇见特殊符号应该去写转义字符比如
              " :&quot;   或者 ''
              && ： &amp;&amp;
         -->
        <if test="name !=null and name !=''">
             name like #{name} and
        </if>
        <!--<if test="address !=null and address !='' ">-->
        <if test="address !=null &amp;&amp; address !=&quot;&quot; ">
             address =#{address}  and
        </if>
        <if test="email !=null and email.trim()!=&quot;&quot;">
             email=#{email} and
        </if>
        <!--ognl会进行字符串与数字的转换判断 "0"==0  字符串0和数字0是一样的 -->
        <if test="gender==0 or gender==1">
             gender=#{gender} and
        </if>
        </trim>
    </select>
```

这样就会自动在最前面添加一个where，然后在最后面去除一个and

#### sql_choose分支选择

类似于java里的 switch   case

```xml
    <!--choose：分支选择
        如果带了id就用id查，如果带了name就用name查，只会进入其中一个
        choose标签里的when标签里的test就是判断条件
    -->
<select id="getEmpsByonditionChoose" resultType="com.zero.bean.Employee">
      select * from tb_user
      <where>
          <choose>
              <when test="id!=null">
                  id=#{id}
              </when>
              <when test="name!=null">
                  name like #{name}
              </when>
              <when test="email!=null">
                  email = #{email}
              </when>
              <otherwise>
                  gender = 0
              </otherwise>
          </choose>
      </where>
</select>
```

如果传入了多个判断条件，那么它只会查符合第一个条件的

#### sql_set

```xml
<!-- 带了那一列的值就更新那一列 -->
<update id="updateEmp">
    update  tb_user
    <trim prefix="set" suffixOverrides=",">
    <if test="name!=null">
        name=#{name}
    </if>
    <if test="email!=null">
        email=#{email}
    </if>
    </trim>
    where id=#{id}
</update>
```

#### sql_foreach遍历集合

```xml
<select id="getEmpByConditionForeach" resultType="com.zero.bean.Employee">
    select * from tb_user where id in
    <!-- collection:指定要遍历的集合：
         list类型的参数会特殊处理封装map中，map的key就叫list
         item：将当前遍历出的元素赋值给指定的定位
         separator:每个元素之间的分隔符
         open:遍历所有结果拼接一个开始的字符
         close:遍历出所有结果拼接一个结束的字符
         index:索引。遍历list的时候是索引，item就是当前值
                    遍历map的时候index表示的就是map的key，item就是map的值
         #{变量名}就能取出变量的值也就是当前遍历出的元素
     -->
    <foreach collection="list" item="item_id" separator=","  open ="(" close =")">
        #{item_id}
    </foreach>
</select>
```

```java
public List<Employee> getEmpByConditionForeach(List<Integer> ids);
```

```java
EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
List<Employee> foreach = mapper.getEmpByConditionForeach(Arrays.asList(4,5,6));
for (Employee e:foreach){
    System.out.println(e);
}
```

##### 批量保存

```xml
<insert id="addEmps" >
    insert into tb_user(name,address,email,gender,d_id)
    values
    <foreach collection="list" item="emp" separator=",">
        (#{emp.name},#{emp.address},#{emp.email},#{emp.gender},#{emp.dept.id})
    </foreach>
</insert>
```

```java
EmployeeMapperDynamicSQL mapper = session.getMapper(EmployeeMapperDynamicSQL.class);
List<Employee> emps = new ArrayList<>();
emps.add(new Employee(null,"smith","beijing","whyathere@163.com","1",new Department(1)));
emps.add(new Employee(null,"smith","beijing","whyathere@163.com","1",new Department(2)));
mapper.addEmps(emps);
session.commit();
```

**另外一种方式**：

```xml
<insert id="addEmps" >
    <foreach collection="list" item="emp" separator=";">
        insert into tb_user(name,address,gender,email,d_id)
        values (#{emp.name},#{emp.address},#{emp.gender},#{emp.email},#{emp.dept.id})
    </foreach>
</insert>
```

但是这个时候会报错。sql不支持。需要进行配置

allowMultiQueries：在一条语句中，运行使用";"来分隔多条查询(真/假，默认值为"假")

#### 内置参数_parameter&_databaseld

```xml
<!--两个内置参数
    不只是方法传递过来的参数可以被用来判断，取值
    mybatis默认还有两个内置参数：
    _parameter:代表整个参数
    如果是单个参数:_parameter就是这个参数；
         多个参数:就会被封装为一个map，_parameter就代表这个map；
    _databaseId:
         如果配置了databaseIdProvider标签
         _databaseId就是代表当前数据库的别名
-->
```

![](https://github.com/whyathere/tuchuang/blob/master/Java%E5%9F%BA%E7%A1%80/databaseProvider.png?raw=true)

![](https://github.com/whyathere/tuchuang/blob/master/Java%E5%9F%BA%E7%A1%80/select.png?raw=true)

#### sql_bing绑定

bing：可以将ognl表达式的值绑定到一个变量中，方便后面引用这个变量的值

```xml
<!-- bind: 可以将OGNL表达式的值绑定到一个变量中，方便后面来引用这个变量的值
     value:就是要绑定值
-->

<bind name="_name" value="'%'+name+'%'"/>
```

上面的是将name拼串模糊查询的样例，然后赋值给_name

#### 抽取可重用sql片段 <sql/>

```xml
<sql id="insertColum">
  user_id,name,email
</sql>
```

引用

```xml
<include refid="insertColum">
	<property name="testColomn" value="adc"/>
</include>
设置了property就可以用${testColomn}取值
```

作用：

1. sql抽取：将常要查询的列名，或者插入用的列名抽取出来方便使用

2. include还可以自定义一些property，sql标签内部就能使用自定义的属性

   include标签的property设置的值，需要用${取值}而不能用#{}

### 缓存

Mybatis包含一个非常强大的查询缓存特性，它可以非常方便地配置和定制。缓存可以极大的提升查询效率。

1. Mybatis系统中默认定义了两级缓存。
2. 一级缓存**和 **二级缓存**

3. 默认情况下，只有一级缓存(SqlSession级别的缓存，也称为本地缓存)开启。

4. 二级缓存需要手动开启和配置，他是基于namespace级别的缓存

5. 为了提高扩展性。Mybatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

#### 一级缓存

一级缓存：本地缓存

与数据库同一次会话期间查询到的数据会放在本地缓存中。以后如果要获取相同的数据，直接从缓存中拿，没必要再去查询数据库。

```java
EmployeeMapperPlus mapper = session.getMapper(EmployeeMapperPlus.class);
Employee emp1 = mapper.getEmployeeById(5);
Employee emp2 = mapper.getEmployeeById(5);
System.out.println(emp1);
System.out.println(emp2);
```

二级缓存：全局缓存

例如：按照id来查询同一个id两次，那么两次拿到的东西其实是一样的。包括对象。是从一级缓存中拿的。

##### 一级缓存失效的四种情况

一级缓存是一直开启的，个人是无法关闭的。

一级缓存是sqlSession级别的缓存。

一级缓存失效情况：一级缓存没有使用，效果是还需要再向数据库发送查询

1. sqlsession不同

2. ```java
   EmployeeMapperPlus mapper = session.getMapper(EmployeeMapperPlus.class);
   Employee emp1 = mapper.getEmployeeById(5);
   System.out.println(emp1);
   
   SqlSession session1 = sessionFactory.openSession();
   EmployeeMapperPlus mapper1 = session1.getMapper(EmployeeMapperPlus.class);
   Employee employeeById = mapper1.getEmployeeById(5);
   System.out.println(employeeById);
   ```

   两次结果就不同了。而且发送了两次sql查询语句。

3. sqlSession相同，但是查询条件不同，当前这个session还没有这个数据

4. sqlSession相同，两次查询之间执行了增删改操作。添加了信息就会重新发送

5. sqlSession相同，但是手动清除了一级缓存。

openSession.clearCache();    //清除缓存

#### 二级缓存介绍

二级缓存是基于namespace级别的缓存：一个名称空间对应一个二级缓存

二级缓存工作机制：

1. 一个会话查询一条数据：这个数据就会被放在当前会话的一级缓存中；
2. 如果当前会话关闭了，这个会话对应的一级缓存也就没有了。但是mybatis并没有清空，如果会话关闭，一级缓存中的数据会被保存到二级缓存中。新的会话查询信息，就可以参照二级缓存
3. sqlSession既有employeemapper查的employee又有DepartmentMapper查的Department对象。会被放在两个不同的namespace中，会放在自己的map中。

#### 二级缓存的使用

使用：开启全局二级缓存配置

1. 

```xml
<!--显式的指定每个我们需要更改的配置的值，即使他是默认的。防止版本更新带来的问题-->
<setting name="cacheEnabled" value="true"/>
```

2.

由于二级缓存是基于mapper级别的，一个namespace对应一个二级缓存

需要去mapper.xml中配置二级缓存。

```xml
<mapper namespace="com.zero.dao.EmployeeMapper">
    <cache></cache>
```

这样即开启了。

但是也有一些默认的策略。

```xml
<cache eviction="" flushInterval="" readOnly="" size="" type=""></cache>
```

**eviction**：缓存的回收策略:

- LRU - 最近最少使用的；移除最长时间不被使用的对象。

- FIFO - 先进先出；按照对象进入缓存的顺序来移除它们
- SOFT - 软引用;移除基于垃圾回收器状态和软引用规则的对象
- WEAK - 弱引用；更积极地移除基于垃圾收集器状态和弱引用规则的对象
- 默认的是LRU。

**flushInterval** : 缓存刷新间隔

缓存多长时间清空一次，默认是不清空，设置一个毫秒值

**readOnly**：缓存是否只读。

true:只读。mybatis认为所有从缓存中获取数据的操作都是只读操作，不会修改操作

​		mybatis为了加快获取速度，直接就会将数据在缓存中的引用交给用户。不安全，但是速度快

false：非只读；mybatis觉的获取来的数据可能会被修改。

​			mybatis会利用序列化&反序列化技术克隆一份新的数据给你。安全，速度稍慢一点。

默认是false

**size**：缓存存放多少元素

**type**:"",指定自定义缓存的全类名；

​		实现Cache接口即可。

```xml
<cache eviction="FOFO" flushInterval="60000" readOnly="false" size="1024" type=""></cache>
```

3. 我们的POJO需要实现序列号接口，因为需要序列化和序列化

首先在未开启二级缓存的时候进行测试

```java
@Test
public void test04() throws IOException {
    String resource = "mybatis/mybatis-config.xml";
    InputStream is = Resources.getResourceAsStream(resource);
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
    SqlSession session1 = sessionFactory.openSession();
    SqlSession session2 = sessionFactory.openSession();
    try {
        EmployeeMapper mapper1 = session1.getMapper(EmployeeMapper.class);
        EmployeeMapper mapper2 = session2.getMapper(EmployeeMapper.class);
        Employee employee = mapper1.getEmployee(5);
        System.out.println(employee);
        session1.close();
        Employee employee1 = mapper2.getEmployee(5);
        System.out.println(employee1);
        session2.close();
    } finally {
        session1.close();
        session2.close();
    }
}
```

此时会进行两次sql查询。

```xml
<cache eviction="FIFO" flushInterval="60000" readOnly="false" size="1024"></cache>
```

开启缓存

就只会查询一次了，这里要注意一定要实现Employee类的序列化。

这次查询 是从二级缓存中获取，而不是再去查询。

虽然关闭了一级缓存，但是放到了二级缓存中。

只有会话提交或者关闭，一级缓存中的数据才会转移到二级缓存中。

#### 缓存有关的设置以及属性

1. 开启缓存，false的时候是关闭缓存。关闭的是二级缓存，一级缓存一直可用

2. ```xml
   <setting name="cacheEnabled" value="true"/>
   ```

3. 为什么可以用缓存。

   在每一个查询<select/>标签都会有一个属性，userCache使用缓存，默认为true

   一级二级都可以使用，当关闭的时候禁用的时候一级缓存依然可以使用。

   useCache对一级缓存是没有影响的，影响的是二级缓存。

4. 每个增删改标签都有一个属性 flushCache = "true"；增删改执行完成后就会清除缓存。清除一级缓存和二级缓存。都会清除。虽然会从二级缓存查询，但是二级缓存已经被清除了。查询是不清除缓存的，如果查询标签设置flushCache = "true"也会禁用掉一级二级缓存。

5. sqlSession.clearCache();  只是清除Session的一级缓存跟二级缓存没有关系

   #### 缓存的原理总结

   一级缓存会话关闭才会保存在二级缓存中。

   关闭以后，新的会话进来，已经存在二级缓存的话就会先查找二级缓存中是否有对应的数据。

   一进来就先看二级缓存，因为它的范围更大，如果没有才会去查找一级缓存。最好才会去查找数据库。

### 整合ehcache

1、首先导入依赖

```xml
<dependency>
   <groupId>org.mybatis.caches</groupId>
   <artifactId>mybatis-ehcache</artifactId>
   <version>1.1.0</version>
</dependency>
```

然后在mapper中添加一个标签就可以了

<cache type="org.mybatis.caches.encache.EncacheCache" />

ehcache运行的时候需要一个ehcache.xml文件

### Mybatis语句

```sql
    <sql id="Base_Column_List" >
        `id`,`user_id`,`channel_id`,`create_user`,`update_user`,`create_time`,`update_time`,`status`
    </sql>
```

``是为了防止关键字的，一般最好加上























