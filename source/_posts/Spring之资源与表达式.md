---
title: Spring之资源与表达式
date: 2018-05-24 11:27:18
tags: Spring
categories: 框架
---

![](https://github.com/whyathere/tuchuang/blob/master/%E7%BE%8E%E5%9B%BE/%E6%9C%88%E5%85%89%E4%B8%8B%E7%9A%84%E7%A5%81%E8%BF%9E%E5%B1%B1%E6%9C%80%E9%AB%98%E5%B3%B0.jpg?raw=true)

抄自：开涛  http://jinnianshilongnian.iteye.com/blog/1416319

### 

## 资源

### 资源基础知识

#### 概述

在日常程序开发中，处理外部资源是很繁琐的事情，我们可能需要处理URL资源、File资源、ClassPath相关资源、服务器相关资源(JBoss AS 5.x 上的VFS资源)等等很多资源。因此处理这些资源需要使用不同的接口，这就增加了我们系统的复杂性；而且处理这些资源步骤都是类似的(打开资源、读取资源、关闭资源)，因此如果能抽象出一个统一的接口来对这些底层资源进行统一访问，是不是很方便，而且使我们系统更加简洁，都是对不同的底层资源使用同一个接口进行访问。

Spring提供了一个Resource接口来统一这些底层资源一直的访问，而且提供了一些便利的接口，从而能提供我们的生产力。

#### Resource接口

Spring的Resource接口代表底层外部资源，提供了对底层外部资源的一致性访问接口。

```java
public interface InputStreamSource {  
    InputStream getInputStream() throws IOException;  
}  
```

1)InputStreamSource接口解析： 

**getInputStream：**每次调用都将返回一个新鲜的资源对应的java.io.InputStream字节流，调用者在使用完毕后必须关闭该资源。

2）Resource接口继承InputStreamSource接口，并提供一些便利方法：

- **exists：**返回当前Resource代表的底层资源是否可读，true表示可读

- **isReadable**：返回当前Resource代表的底层资源是否可读，true表示可读。

-  **isOpen**：返回当前Resource代表的底层资源是否已经打开，如果返回true，则只能被读取一次然后关闭以避免资源泄露；常见的Resource实现一般返回false。 

-   **getURL**：如果当前Resource代表的底层资源能由java.util.URL代表，则返回该URL，否则抛出IOException。

- **getURI**：如果当前Resource代表的底层资源能由java.util.URI代表，则返回该URI，否则抛出IOException。 

- **getFile**：如果当前Resource代表的底层资源能由java.io.File代表，则返回该File，否则抛出IOException。 

- **contentLength**：返回当前Resource代表的底层文件资源的长度，一般是值代表的文件资源的长度。

-  **lastModified** ：返回当前Resource代表的底层资源的最后修改时间。 

- **createRelative**：用于创建相对于当前Resource代表的底层资源的资源，比如当前Resource代表文件资源“d:/test/”则createRelative（“test.txt”）将返回表文件资源“d:/test/test.txt”Resource资源。 

-   **getFilename**：返回当前Resource代表的底层文件资源的文件路径，比如File资源“file://d:/test.txt”将返回“d:/test.txt”，而URL资源http://www.javass.cn将返回“”，因为只返回文件路径。 

-  **getDescription**：返回当前Resource代表的底层资源的描述符，通常就是资源的全路径（实际文件名或实际URL地址）。

  Resource接口提供了足够的抽象，足够满足我们日常使用。而且提供了很多内置Resource实现：ByteArrayResource、InputStreamResource 、FileSystemResource 、UrlResource 、ClassPathResource、ServletContextResource、VfsResource等。

  

  

  

  ### 内置Resource实现

  ####  ByteArrayResource



ByteArrayResource代表byte[]数组资源对于“getInputStream”操作将返回一个ByteArrayInputStream。

首先让我们看下使用ByteArrayResource如何处理byte数组资源：

 ```java
public class ResourceTest {

    @Test
    public void testByteArrayResource(){
        Resource resource = new ByteArrayResource("Hello World !".getBytes()); //得到一个操作系统默认的编码格式的字节数组。
        if (resource.exists()){
            dumpStream(resource);
        }
    }
 ```

是不是很简单，让我们看下“dumpStream”实现：

 ```java
private void dumpStream(Resource resource) {
        InputStream is = null;
        try {
            //1.获取文件资源
            is = resource.getInputStream();
            //2.读取资源
            byte[] descBytes = new byte[is.available()];
            is.read(descBytes);
            System.out.println(new String(descBytes));
        } catch (IOException e) {
            e.printStackTrace();
        }
        finally {
            try {
                //3.关闭资源
                is.close();
            } catch (IOException e) {
            }
        }
    }
 ```

​    让我们来仔细看一下代码，dumpStream方法很抽象定义了访问流的三部曲：打开资源、读取资源、关闭资源，所以dunpStrean可以再进行抽象从而能在自己项目中使用；byteArrayResourceTest测试方法，也定义了基本步骤：定义资源、验证资源存在、访问资源。

​        ByteArrayResource可多次读取数组资源，即isOpen ()永远返回false。

####  InputStreamResource

​       InputStreamResource代表java.io.InputStream字节流，对于“getInputStream ”操作将直接返回该字节流，因此只能读取一次该字节流，即“isOpen”永远返回true。

​        让我们看下测试代码吧：

 ```java
    @Test
    public void testInputStreamResource() {
        ByteArrayInputStream bis = new ByteArrayInputStream("Hello World!".getBytes());
        Resource resource = new InputStreamResource(bis);
        if(resource.exists()) {
            dumpStream(resource);
        }
        Assert.assertEquals(true, resource.isOpen());
    }
 ```

#### FileSystemResource

​       FileSystemResource代表java.io.File资源，对于“getInputStream ”操作将返回底层文件的字节流，“isOpen”将永远返回false，从而表示可多次读取底层文件的字节流。

​        让我们看下测试代码吧：

 ```java

    @Test
    public void testFileResource() {
        File file = new File("d:/test.txt");
        Resource resource = new FileSystemResource(file);
        if(resource.exists()) {
            dumpStream(resource);
        }
        Assert.assertEquals(false, resource.isOpen());
    }
 ```

​       注意由于“isOpen”将永远返回false，所以可以多次调用dumpStream(resource)。

####  ClassPathResource

​       ClassPathResource代表classpath路径的资源，将使用ClassLoader进行加载资源。classpath 资源存在于类路径中的文件系统中或jar包里，且“isOpen”永远返回false，表示可多次读取资源。

 	       ClassPathResource加载资源替代了Class类和ClassLoader类的“getResource(String name)”和“getResourceAsStream(String name)”两个加载类路径资源方法，提供一致的访问方式。

 ClassPathResource提供了三个构造器：

​          **public ClassPathResource(String path)**：使用默认的ClassLoader加载“path”类路径资源；

​          **public ClassPathResource(String path, ClassLoader classLoader)**：使用指定的ClassLoader加载“path”类路径资源；

 比如当前类路径是“cn.javass.spring.chapter4.ResourceTest”，而需要加载的资源路径是“cn/javass/spring/chapter4/test1.properties”，则将加载的资源在“cn/javass/spring/chapter4/test1.properties”；

​          **public ClassPathResource(String path, Class<?> clazz)**：使用指定的类加载“path”类路径资源，将加载相对于当前类的路径的资源；

 比如当前类路径是“cn.javass.spring.chapter4.ResourceTest”，而需要加载的资源路径是“cn/javass/spring/chapter4/test1.properties”，则将加载的资源在“cn/javass/spring/chapter4/cn/javass/spring/chapter4/test1.properties”；

​        而如果需要 加载的资源路径为“test1.properties”，将加载的资源为“cn/javass/spring/chapter4/test1.properties”。

​        让我们直接看测试代码吧：

 1）使用默认的加载器加载资源，将加载当前ClassLoader类路径上相对于根路径的资源：

 ```java
    @Test
    public void testClasspathResourceByDefaultClassLoader() throws IOException {
        Resource resource = new ClassPathResource("cn/javass/spring/chapter4/test1.properties");
        if(resource.exists()) {
            dumpStream(resource);
        }
        System.out.println("path:" + resource.getFile().getAbsolutePath());
        Assert.assertEquals(false, resource.isOpen());
    }
 ```

2）使用指定的ClassLoader进行加载资源，将加载指定的ClassLoader类路径上相对于根路径的资源：

 ```java
    @Test
    public void testClasspathResourceByClassLoader() throws IOException {
        ClassLoader cl = this.getClass().getClassLoader();
        Resource resource = new ClassPathResource("cn/javass/spring/chapter4/test1.properties" , cl);
        if(resource.exists()) {
            dumpStream(resource);
        }
        System.out.println("path:" + resource.getFile().getAbsolutePath());
        Assert.assertEquals(false, resource.isOpen());
    }
 ```

3）使用指定的类进行加载资源，将尝试加载相对于当前类的路径的资源：

 ```java
    @Test
    public void testClasspathResourceByClass() throws IOException {
        Class clazz = this.getClass();
        Resource resource1 = new ClassPathResource("cn/javass/spring/chapter4/test1.properties" , clazz);
        if(resource1.exists()) {
            dumpStream(resource1);
        }
        System.out.println("path:" + resource1.getFile().getAbsolutePath());
        Assert.assertEquals(false, resource1.isOpen());

        Resource resource2 = new ClassPathResource("test1.properties" , this.getClass());
        if(resource2.exists()) {
            dumpStream(resource2);
        }
        System.out.println("path:" + resource2.getFile().getAbsolutePath());
        Assert.assertEquals(false, resource2.isOpen());
    }
 ```

​       “resource1”将加载cn/javass/spring/chapter4/cn/javass/spring/chapter4/test1.properties资源；“resource2”将加载“cn/javass/spring/chapter4/test1.properties”；

 4）加载jar包里的资源，首先在当前类路径下找不到，最后才到Jar包里找，而且在第一个Jar包里找到的将被返回：

 ```java
    @Test
    public void classpathResourceTestFromJar() throws IOException {
        Resource resource = new ClassPathResource("overview.html");
        if(resource.exists()) {
            dumpStream(resource);
        }
        System.out.println("path:" + resource.getURL().getPath());
        Assert.assertEquals(false, resource.isOpen());
    }
 ```

如果当前类路径包含“overview.html”，在项目的“resources”目录下，将加载该资源，否则将加载Jar包里的“overview.html”，而且不能使用“resource.getFile()”，应该使用“resource.getURL()”，因为资源不存在于文件系统而是存在于jar包里，URL类似于“file:/C:/.../***.jar!/overview.html”。

 类路径一般都是相对路径，即相对于类路径或相对于当前类的路径，因此如果使用“/test1.properties”带前缀“/”的路径，将自动删除“/”得到“test1.properties”。

####  UrlResource

​       UrlResource代表URL资源，用于简化URL资源访问。“isOpen”永远返回false，表示可多次读取资源。

​        UrlResource一般支持如下资源访问：

​      UrlResource一般支持如下资源访问：

​         **http：**通过标准的http协议访问web资源，如new UrlResource(“http://地址”)；

​         **ftp：**通过ftp协议访问资源，如new UrlResource(“ftp://地址”)；

​         **file：**通过file协议访问本地文件系统资源，如new UrlResource(“file:d:/test.txt”)；

具体使用方法在此就不演示了，可以参考cn.javass.spring.chapter4.ResourceTest中urlResourceTest测试方法。

#### ServletContextResource

​       ServletContextResource代表web应用资源，用于简化servlet容器的ServletContext接口的getResource操作和getResourceAsStream操作；在此就不具体演示了。

####  VfsResource

VfsResource代表Jboss 虚拟文件系统资源。

 Jboss VFS(Virtual File System)框架是一个文件系统资源访问的抽象层，它能一致的访问物理文件系统、jar资源、zip资源、war资源等，VFS能把这些资源一致的映射到一个目录上，访问它们就像访问物理文件资源一样，而其实这些资源不存在于物理文件系统。

 在示例之前需要准备一些jar包，在此我们使用的是Jboss VFS3版本，可以下载最新的Jboss AS 6x，拷贝lib目录下的“jboss-logging.jar”和“jboss-vfs.jar”两个jar包拷贝到我们项目的lib目录中并添加到“Java Build Path”中的“Libaries”中。

 让我们看下示例（cn.javass.spring.chapter4.ResourceTest）：

 ```java
 @Test
    public void testVfsResourceForRealFileSystem() throws IOException {
//1.创建一个虚拟的文件目录
        VirtualFile home = VFS.getChild("/home");
//2.将虚拟目录映射到物理的目录
        VFS.mount(home, new RealFileSystem(new File("d:")));
//3.通过虚拟目录获取文件资源
        VirtualFile testFile = home.getChild("test.txt");
//4.通过一致的接口访问
        Resource resource = new VfsResource(testFile);
        if(resource.exists()) {
            dumpStream(resource);
        }
        System.out.println("path:" + resource.getFile().getAbsolutePath());
        Assert.assertEquals(false, resource.isOpen());
    }
    @Test
    public void testVfsResourceForJar() throws IOException {
//1.首先获取jar包路径
        File realFile = new File("lib/org.springframework.beans-3.0.5.RELEASE.jar");
        //2.创建一个虚拟的文件目录
        VirtualFile home = VFS.getChild("/home2");
        //3.将虚拟目录映射到物理的目录
        VFS.mountZipExpanded(realFile, home,
                TempFileProvider.create("tmp", Executors.newScheduledThreadPool(1)));
//4.通过虚拟目录获取文件资源
        VirtualFile testFile = home.getChild("META-INF/spring.handlers");
        Resource resource = new VfsResource(testFile);
        if(resource.exists()) {
            dumpStream(resource);
        }
        System.out.println("path:" + resource.getFile().getAbsolutePath());
        Assert.assertEquals(false, resource.isOpen());
    }

 ```

​       通过VFS，对于jar里的资源和物理文件系统访问都具有一致性，此处只是简单示例，如果需要请到Jboss官网深入学习。

 

### 访问Resource

#### ResourceLoader接口

ResourceLoader接口用于返回Resource对象；其实现可以看作是一个生产Resource的工厂类。

 ```java
    public interface ResourceLoader {
        Resource getResource(String location);
        ClassLoader getClassLoader();
    }
 ```

​       getResource接口用于根据提供的location参数返回相应的Resource对象；而getClassLoader则返回加载这些Resource的ClassLoader。

​        Spring提供了一个适用于所有环境的DefaultResourceLoader实现，可以返回ClassPathResource、UrlResource；还提供一个用于web环境的ServletContextResourceLoader，它继承了DefaultResourceLoader的所有功能，又额外提供了获取ServletContextResource的支持。

​        ResourceLoader在进行加载资源时需要使用前缀来指定需要加载：“classpath:path”表示返回ClasspathResource，“http://path”和“file:path”表示返回UrlResource资源，如果不加前缀则需要根据当前上下文来决定，DefaultResourceLoader默认实现可以加载classpath资源，如代码所示（cn.javass.spring.chapter4.ResourceLoaderTest）：

```java
    @Test
    public void testResourceLoad() {
        ResourceLoader loader = new DefaultResourceLoader();
        Resource resource = loader.getResource("classpath:cn/javass/spring/chapter4/test1.txt");
        //验证返回的是ClassPathResource
        Assert.assertEquals(ClassPathResource.class, resource.getClass());
        Resource resource2 = loader.getResource("file:cn/javass/spring/chapter4/test1.txt");
        //验证返回的是ClassPathResource
        Assert.assertEquals(UrlResource.class, resource2.getClass());
        Resource resource3 = loader.getResource("cn/javass/spring/chapter4/test1.txt");
        //验证返默认可以加载ClasspathResource
        Assert.assertTrue(resource3 instanceof ClassPathResource);
    }
```

对于目前所有ApplicationContext都实现了ResourceLoader，因此可以使用其来加载资源。

​         **ClassPathXmlApplicationContext：**不指定前缀将返回默认的ClassPathResource资源，否则将根据前缀来加载资源；

​         **FileSystemXmlApplicationContext：**不指定前缀将返回FileSystemResource，否则将根据前缀来加载资源；

​         **WebApplicationContext：**不指定前缀将返回ServletContextResource，否则将根据前缀来加载资源；

​         **其他：**不指定前缀根据当前上下文返回Resource实现，否则将根据前缀来加载资源。

####  ResourceLoaderAware接口

​       ResourceLoaderAware是一个标记接口，用于通过ApplicationContext上下文注入ResourceLoader。

 ```java
    public interface ResourceLoaderAware {
        void setResourceLoader(ResourceLoader resourceLoader);
    }
 ```

1）  首先准备测试Bean，我们的测试Bean还简单只需实现ResourceLoaderAware接口，然后通过回调将ResourceLoader保存下来就可以了：

```java
public class ResourceBean implements ResourceLoaderAware {

    private ResourceLoader resourceLoader;

    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }
    public ResourceLoader getResourceLoader() {
        return resourceLoader;
    }
}
```

2）  配置Bean定义（chapter4/resourceLoaderAware.xml）：

 ```xml
    <bean class="com.zero.resource.ResourceBean"/>
 ```

3）测试(cn.javass.spring.chapter4.ResoureLoaderAwareTest)：

 ```java
public class HelloTest {
    @Test
    public void testHelloWorld() {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("helloworld.xml");
        ResourceBean resourceBean = ctx.getBean(ResourceBean.class);
        ResourceLoader loader = resourceBean.getResourceLoader();
        Assert.assertTrue(loader instanceof ApplicationContext);
    }
}
 ```

​       注意此处“loader instanceof ApplicationContext”，说明了ApplicationContext就是个ResoureLoader。

​        由于上述实现回调接口注入ResourceLoader的方式属于侵入式，所以不推荐上述方法，可以采用更好的自动注入方式，如“byType”和“constructor”，此处就不演示了。   

####  注入Resource

​       通过回调或注入方式注入“ResourceLoader”，然后再通过“ResourceLoader”再来加载需要的资源对于只需要加载某个固定的资源是不是很麻烦，有没有更好的方法类似于前边实例中注入“java.io.File”类似方式呢？

​        Spring提供了一个PropertyEditor “ResourceEditor”用于在注入的字符串和Resource之间进行转换。因此可以使用注入方式注入Resource。

​        ResourceEditor完全使用ApplicationContext根据注入的路径字符串获取相应的Resource，说白了还是自己做还是容器帮你做的问题。

 接下让我们看下示例：

​        1）准备Bean：

 ```java
import org.springframework.core.io.Resource;
public class ResourceBean3 {
    private Resource resource;
    public Resource getResource() {
        return resource;
    }
    public void setResource(Resource resource) {
        this.resource = resource;
    }
}  
 ```

​       2）准备配置文件（chapter4/ resourceInject.xml）：

 ```xml
    <bean id="resourceBean1" class="com.zero.resource.ResourceBean3">
        <property name="resource" value="cn/javass/spring/chapter4/test1.properties"/>
    </bean>
    <bean id="resourceBean2" class="com.zero.resource.ResourceBean3">
        <property name="resource"
                  value="classpath:cn/javass/spring/chapter4/test1.properties"/>
    </bean>
 ```

​       注意此处“resourceBean1”注入的路径没有前缀表示根据使用的ApplicationContext实现进行选择Resource实现。

​        3）让我们来看下测试代码（cn.javass.spring.chapter4.ResourceInjectTest）吧：

 ```java
    @Test
    public void test() {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("chapter4/resourceInject.xml");
        ResourceBean3 resourceBean1 = ctx.getBean("resourceBean1", ResourceBean3.class);
        ResourceBean3 resourceBean2 = ctx.getBean("resourceBean2", ResourceBean3.class);
        Assert.assertTrue(resourceBean1.getResource() instanceof ClassPathResource);
        Assert.assertTrue(resourceBean2.getResource() instanceof ClassPathResource);
    }
 ```

​       接下来一节让我们深入ApplicationContext对各种Resource的支持，及如何使用更便利的资源加载方式。

###  Resource通配符路径

#### 使用路径通配符加载Resource

​        前面介绍的资源路径都是非常简单的一个路径匹配一个资源，Spring还提供了一种更强大的Ant模式通配符匹配，从能一个路径匹配一批资源。

​        Ant路径通配符支持“？”、“*”、“**”，注意通配符匹配不包括目录分隔符“/”：

 **“?****”：匹配一个字符**，如“config?.xml”将匹配“config1.xml”；

​         **“\*****”：匹配零个或多个字符串**，如“cn/*/config.xml”将匹配“cn/javass/config.xml”，但不匹配匹配“cn/config.xml”；而“cn/config-*.xml”将匹配“cn/config-dao.xml”；

​         **“\******”：匹配路径中的零个或多个目录**，如“cn/**/config.xml”将匹配“cn /config.xml”，也匹配“cn/javass/spring/config.xml”；而“cn/javass/config-**.xml”将匹配“cn/javass/config-dao.xml”，即把“**”当做两个“*”处理。

 

Spring提供AntPathMatcher来进行Ant风格的路径匹配。具体测试请参考cn.javass.spring.chapter4. AntPathMatcherTest。

 

Spring在加载类路径资源时除了提供前缀“classpath:”的来支持加载一个Resource，还提供一个前缀“classpath*:”来支持加载所有匹配的类路径Resource。

 

Spring提供ResourcePatternResolver接口来加载多个Resource，该接口继承了ResourceLoader并添加了“Resource[] getResources(String locationPattern)”用来加载多个Resource：

```java
public interface ResourcePatternResolver extends ResourceLoader {  
       String CLASSPATH_ALL_URL_PREFIX = "classpath*:";  
       Resource[] getResources(String locationPattern) throws IOException;  
}  
```

Spring提供了一个ResourcePatternResolver实现PathMatchingResourcePatternResolver，它是基于模式匹配的，默认使用AntPathMatcher进行路径匹配，它除了支持ResourceLoader支持的前缀外，还额外支持“classpath*:”用于加载所有匹配的类路径Resource，ResourceLoader不支持前缀“classpath*:”：

 首先做下准备工作，在项目的“resources”创建“META-INF”目录，然后在其下创建一个“INDEX.LIST”文件。同时在“org.springframework.beans-3.0.5.RELEASE.jar”和“org.springframework.context-3.0.5.RELEASE.jar”两个jar包里也存在相同目录和文件。然后创建一个“LICENSE”文件，该文件存在于“com.springsource.cn.sf.cglib-2.2.0.jar”里。

 **一、“classpath**”：** 用于加载类路径（包括jar包）中的一个且仅一个资源；对于多个匹配的也只返回一个，所以如果需要多个匹配的请考虑“classpath*:”前缀；

 ```java
@Test  
public void testClasspathPrefix() throws IOException {  
    ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();  
    //只加载一个绝对匹配Resource，且通过ResourceLoader.getResource进行加载  
    Resource[] resources=resolver.getResources("classpath:META-INF/INDEX.LIST");  
    Assert.assertEquals(1, resources.length);  
    //只加载一个匹配的Resource，且通过ResourceLoader.getResource进行加载  
    resources = resolver.getResources("classpath:META-INF/*.LIST");  
    Assert.assertTrue(resources.length == 1);             
}  
 ```

**二、“classpath\*”：** 用于加载类路径（包括jar包）中的所有匹配的资源。带通配符的classpath使用“ClassLoader”的“Enumeration<URL> getResources(String name)”方法来查找通配符之前的资源，然后通过模式匹配来获取匹配的资源。如“classpath:META-INF/*.LIST”将首先加载通配符之前的目录“META-INF”，然后再遍历路径进行子路径匹配从而获取匹配的资源。 

```java
@Test  
public void testClasspathAsteriskPrefix () throws IOException {  
     ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();        
     //将加载多个绝对匹配的所有Resource  
    //将首先通过ClassLoader.getResources("META-INF")加载非模式路径部分  
    //然后进行遍历模式匹配  
    Resource[] resources=resolver.getResources("classpath*:META-INF/INDEX.LIST");  
    Assert.assertTrue(resources.length > 1);      
    //将加载多个模式匹配的Resource  
    resources = resolver.getResources("classpath*:META-INF/*.LIST");  
    Assert.assertTrue(resources.length > 1);    
}  
```

注意“resources.length >1”说明返回多个Resource。不管模式匹配还是非模式匹配只要匹配的都将返回。

        在“com.springsource.cn.sf.cglib-2.2.0.jar”里包含“asm-license.txt”文件，对于使用“classpath*: asm-*.txt”进行通配符方式加载资源将什么也加载不了“asm-license.txt”文件，注意一定是模式路径匹配才会遇到这种问题。这是由于“ClassLoader”的“getResources(String name)”方法的限制，对于name为“”的情况将只返回文件系统的类路径，不会包换jar包根路径。 

```java
@Test  
public void testClasspathAsteriskPrefixLimit() throws IOException {  
    ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();      //将首先通过ClassLoader.getResources("")加载目录，  
    //将只返回文件系统的类路径不返回jar的跟路径  
    //然后进行遍历模式匹配  
    Resource[] resources = resolver.getResources("classpath*:asm-*.txt");  
    Assert.assertTrue(resources.length == 0);  
    //将通过ClassLoader.getResources("asm-license.txt")加载  
    //asm-license.txt存在于com.springsource.net.sf.cglib-2.2.0.jar  
    resources = resolver.getResources("classpath*:asm-license.txt");  
    Assert.assertTrue(resources.length > 0);       
    //将只加载文件系统类路径匹配的Resource  
    resources = resolver.getResources("classpath*:LICENS*");  
    Assert.assertTrue(resources.length == 1);  
}  
```

对于“resolver.getResources("classpath*:asm-*.txt");”，由于在项目“resources”目录下没有所以应该返回0个资源；“resolver.getResources("classpath*:asm-license.txt");”将返回jar包里的Resource；“resolver.getResources("classpath*:LICENS*");”，因为将只返回文件系统类路径资源，所以返回1个资源。

 因此加载通配符路径时（即路径中包含通配符），必须包含一个根目录才能保证加载的资源是所有的，而不是部分。

 **三、“file”：**加载一个或多个文件系统中的Resource。如“file:D:/*.txt”将返回D盘下的所有txt文件；      

 

**四、无前缀**：通过ResourceLoader实现加载一个资源。

 

AppliacationContext提供的getResources方法将获取资源委托给ResourcePatternResolver实现，默认使用PathMatchingResourcePatternResolver。所有在此就无需介绍其使用方法了。



#### 注入Resource数组

​       Spring还支持注入Resource数组，直接看配置如下：

 ```xml
<bean id="resourceBean1" class="cn.javass.spring.chapter4.bean.ResourceBean4">  
<property name="resources">  
        <array>  
            <value>cn/javass/spring/chapter4/test1.properties</value>  
            <value>log4j.xml</value>  
        </array>  
    </property>  
</bean>  
<bean id="resourceBean2" class="cn.javass.spring.chapter4.bean.ResourceBean4">  
<property name="resources" value="classpath*:META-INF/INDEX.LIST"/>  
</bean>  
<bean id="resourceBean3" class="cn.javass.spring.chapter4.bean.ResourceBean4">  
<property name="resources">  
        <array>  
            <value>cn/javass/spring/chapter4/test1.properties</value>  
            <value>classpath*:META-INF/INDEX.LIST</value>  
        </array>  
    </property>  
</bean>  
 ```

​       “resourceBean1”就不用多介绍了，传统实现方式；对于“resourceBean2”则使用前缀“classpath*”，看到这大家应该懂的，加载匹配多个资源；“resourceBean3”是混合使用的；测试代码在“cn.javass.spring.chapter4.ResourceInjectTest.testResourceArrayInject”。

​        Spring通过ResourceArrayPropertyEditor来进行类型转换的，而它又默认使用“PathMatchingResourcePatternResolver”来进行把路径解析为Resource对象。所有大家只要会使用“PathMatchingResourcePatternResolver”，其它一些实现都是委托给它的，比如AppliacationContext的“getResources”方法等。

####  AppliacationContext实现对各种Resource的支持

​       **一、ClassPathXmlApplicationContext：**默认将通过classpath进行加载返回ClassPathResource，提供两类构造器方法：

 ```java
public class ClassPathXmlApplicationContext {  
    //1）通过ResourcePatternResolver实现根据configLocation获取资源  
       public ClassPathXmlApplicationContext(String configLocation);  
       public ClassPathXmlApplicationContext(String... configLocations)；  
       public ClassPathXmlApplicationContext(String[] configLocations, ……);  
        
    //2）通过直接根据path直接返回ClasspathResource  
       public ClassPathXmlApplicationContext(String path, Class clazz);  
       public ClassPathXmlApplicationContext(String[] paths, Class clazz);  
       public ClassPathXmlApplicationContext(String[] paths, Class clazz, ……);  
}  
 ```

​       第一类构造器是根据提供的配置文件路径使用“ResourcePatternResolver ”的“getResources()”接口通过匹配获取资源；即如“classpath:config.xml”

​        第二类构造器则是根据提供的路径和clazz来构造ClassResource资源。即采用“public ClassPathResource(String path, Class<?> clazz)”构造器获取资源。

​        **二、FileSystemXmlApplicationContext：**将加载相对于当前工作目录的“configLocation”位置的资源，注意在linux系统上不管“configLocation”是否带“/”，都作为相对路径；而在window系统上如“D:/resourceInject.xml”是绝对路径。因此在除非很必要的情况下，不建议使用该ApplicationContext。

 ```java
public class FileSystemXmlApplicationContext{  
       public FileSystemXmlApplicationContext(String configLocation);  
       public FileSystemXmlApplicationContext(String... configLocations,……);  
}  
 ```

```java
//windows系统，第一个将相对于当前vm路径进行加载；  
//第二个则是绝对路径方式加载  
new FileSystemXmlApplicationContext("chapter4/config.xml");  
new FileSystemXmlApplicationContext("d:/chapter4/confg.xml"); 
```

​       此处还需要注意：在linux系统上，构造器使用的是相对路径，而ctx.getResource()方法如果以“/”开头则表示获取绝对路径资源，而不带前导“/”将返回相对路径资源。如下：

 ```java
//linux系统，第一个将相对于当前vm路径进行加载；  
//第二个则是绝对路径方式加载  
ctx.getResource ("chapter4/config.xml");  
ctx.getResource ("/root/confg.xml");  
//windows系统，第一个将相对于当前vm路径进行加载；  
//第二个则是绝对路径方式加载  
ctx.getResource ("chapter4/config.xml");  
ctx.getResource ("d:/chapter4/confg.xml");  
 ```

​       因此如果需要加载绝对路径资源最好选择前缀“file”方式，将全部根据绝对路径加载。如在linux系统“ctx.getResource ("file:/root/confg.xml");”    

##  Spring表达式语言

### 概述

#### 简介

​       Spring表达式语言全称为“Spring Expression Language”，缩写为“SpEL”，类似于Struts2x中使用的OGNL表达式语言，能在运行时构建复杂表达式、存取对象图属性、对象方法调用等等，并且能与Spring功能完美整合，如能用来配置Bean定义。

​        表达式语言给静态Java语言增加了动态功能。

​        SpEL是单独模块，只依赖于core模块，不依赖于其他模块，可以单独使用。

####  能干什么

 表达式语言一般是用最简单的形式完成最主要的工作，减少我们的工作量。

​       SpEL支持如下表达式：

**一、基本表达式：**字面量表达式、关系，逻辑与算数运算表达式、字符串连接及截取表达式、三目运算及Elivis表达式、正则表达式、括号优先级表达式；

**二、类相关表达式：**类类型表达式、类实例化、instanceof表达式、变量定义及引用、赋值表达式、自定义函数、对象属性存取及安全导航表达式、对象方法调用、Bean引用；

**三、集合相关表达式：**内联List、内联数组、集合，字典访问、列表，字典，数组修改、集合投影、集合选择；不支持多维内联数组初始化；不支持内联字典定义；

**四、其他表达式**：模板表达式。

**注：SpEL表达式中的关键字是不区分大小写的。**

### SpEL基础

#### HelloWorld

​       首先准备支持SpEL的Jar包：“org.springframework.expression-3.0.5.RELEASE.jar”将其添加到类路径中。

​        SpEL在求表达式值时一般分为四步，其中第三步可选：首先构造一个解析器，其次解析器解析字符串表达式，在此构造上下文，最后根据上下文得到表达式运算后的值。

​        让我们看下代码片段吧：

 ```java
public class SpELTest {
    @Test
    public void helloWorld() {
        ExpressionParser parser = new SpelExpressionParser();
        Expression expression =
                parser.parseExpression("('Hello' + ' World').concat(#end)");
        EvaluationContext context = new StandardEvaluationContext();
        context.setVariable("end", "!");
        Assert.assertEquals("Hello World!", expression.getValue(context));
    }
}
 ```

接下来让我们分析下代码：

1）创建解析器：SpEL使用ExpressionParser接口表示解析器，提供SpelExpressionParser默认实现；

2）解析表达式：使用ExpressionParser的parseExpression来解析相应的表达式为Expression对象。

3）构造上下文：准备比如变量定义等等表达式需要的上下文数据。

4）求值：通过Expression接口的getValue方法根据上下文获得表达式值。



是不是很简单，接下来让我们看下其具体实现及原理吧。

####  SpEL原理及接口

