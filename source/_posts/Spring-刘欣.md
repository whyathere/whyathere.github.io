---
title: Spring刘欣
date: 2018-09-10 17:49:57
tags: Spring
categories: 框架
---

Spring

### 简介

耦合具有两面性。一方面，紧密耦合的代码难以测试，难以复用、难以理解，并且典型地表现出“打地鼠”式的bug特性。另一方面，一定程度的耦合又是必须的—完全没有耦合的代码什么都做不了。为了完成有意义的功能，不同的类必须适当的进行交互。总而言之，耦合是必须的，但应当被小心谨慎地管理。

创建应用组件之间协作的行为通常称为装配（wiring）。Spring有多种装配bean的方式，采用xml是很常见的一种装配行为。

#### 观察它如何工作

Spring通过应用上下文(Application Context)装载bean的定义并把它们组装起来。Spring应用上下文全权负责对象的创建和组装。Spring自带了多种应用上下文的实现，它们之间主要的区别仅仅在于如何配置。

ClassPathXmlApplicationContext	该类加载位于应用程序类路径下的一个或多个XML配置文件。

#### 容纳你的Bean

在基于Spring的应用中，你的应用对象生存于Spring容器(container)中。Spring容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期，从生存到死亡(在这里，可能就是new 到 finalize())。

首先了解容纳对象的容器，有助于理解对象是如何被管理的。

容器上Spring容器的核心。Spring容器使用DI管理构成应用的组建，它会创建相互协作的组件之间的关联。

Spring容器并不是只有一个。Spring自带了多个容器实现，可以归为两种不同的类型。bean工厂上最简单的容器，提供基本的DI支持。应用上下文基于BeanFactory构建，并提供应用框架级别的服务，例如从属性文件解析文本信息以及发布应用事件给感兴趣的事件监听者。

虽然我们可以在bean工厂和应用上下文之间任选一种。但bean工厂对大多数应用来说太低级了，因此，应用上下文要比bean工厂更受欢迎。我们把精力集中在应用上下文上，不再浪费时间讨论bean工厂。

##### 使用应用上下文

常用的几个应用上下文

AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文。

ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。

FileSystemXmlApplicationContext：从文件系统下的一个或多个XML配置文件中加载上下文定义。



DI是组装应用对象的一种方式，借助这种方式对象无需知道依赖来自何处或者依赖的实现方式。不同于自己获取依赖对象，对象会在运行期赋予它们所依赖的对象。依赖对象通常会通过接口了解所注入的对象，这样的话就能确保低耦合。



### 装配Bean

#### Spring 配置的可选方案

Spring容器负责创建应用程序中的bean并通过DI来协调这些对象之间的关系。但是，作为开发人员，你需要告诉Spring要创建哪些bean并且如何将其装配在一起。



### Basic BeanFactory 

SRP:单一职责原则

**职责：**是引起变化的原因

- 如果有多于一个的动机去改变一个类，这个类就具有多于一个职责
- 把多个职责耦合在一起，一个变化可能会消弱或者抑制这个类完成其他职责的能力

**SRP:对于一个类而言，应该仅有一个引起它变化的原因**

```java
Class<?> clz = cl.loadClass(beanClassName);
return clz.newInstance();
```

上面代码能够返回实例的前提是，类有一个缺省的无参构造函数。这样才能用newInstance的方式创建出来。



```java
    @Test
    public void testGetBean(){

        BeanFactory beanFactory = new DefaultBeanFactory("petstore-v1.xml");

        BeanDefinition bd = beanFactory.getBeanDefinition("petStore");

        			        assertEquals("org.litespring.service.test.v1.PetStoreService",bd.getBeanClassName());

        PetStoreService petStore =   (PetStoreService)beanFactory.getBean("petStore");

        assertNotNull(petStore);
    }
```



```java
new DefaultBeanFactory("petstore-v1.xml");
```

能够返回bean的定义，而bean的定义也是一个接口。

然后从factory中获取bean。

实现类放到support下面，接口放到factory下面。

getBean是返回一个实例。







这里面的代码有太多了try/catch

常见的方法引入Exception，包装一个Exception可以throw出去，让调用方去处理。



现在的文件有两个功能，一个是读取xml文件，生成bean定义，就是getBeanDefinition

一个是生成bean的实例。所以要创建两个exception

RuntimeException是一个unchecked的异常。

![](https://i.loli.net/2018/08/31/5b890ff0692d6.jpg)

```java
catch (DocumentException e) {
            throw new BeanDefinitionStoreException("IOException parsing XML document from"+configFile,e);
```

xml异常

```java
        if (bd == null){
            throw new BeanCreationException("Bean Definition does not exist");
//            return null;
        }
```

bean异常





SRP：单一职责原则

- 职责：是引起变化的原因
  - 如果有多于一个的动机起改变一个类，这个类就具有多于一个职责
  - 把多个职责耦合在一起，一个的变化可能会消弱或者抑制这个类完成其他职责的能力
- SRP：对于一个类而言，应该仅有一个引起它变化的原因。

![](https://i.loli.net/2018/09/03/5b8ce0a3bf779.png)

这里现在的DefaultBeanFactory具有多于一个 的职责。

现在需要单独 的抽取出来

BeanDefinition是一个内部的概念，不应该暴露给client也不应该放到BeanFactory这个借口上，只让客户看到一个最小集。

创建一个接口出来，专门用于处理BeanDefinition



创建一个reader来读取并解析xml，那么读取来以后怎么把definition告诉factory呢？让BeanDefiniton提供一个方法，registerBeanDefinition.当reader去解析xml文件后，可以去调用register方法，让reader持有一个BeanFactory的实例。但是这样并不好，BeanFactory是给客户使用的，并不希望客户调用register方法。BeanDefinition本质上是一个内部的概念，把一个xml变成一个BeanDefinition，我们拥有它的意义是通过反射去创建一个Bean的实例。所以BeanDefiniton应该是一个内部的概念。

![](https://i.loli.net/2018/09/03/5b8ce03672ef7.png)

这样有两个好处，1/xmlBeanDefinitionReader只知道一个知识，我可以去获得和注册BeanDefinition，但是并不知道getBean。这个类知道越少的类越好。接口最小化

```
先修改testCase
```



XmlBeanDefinitionReader设置单一职责

```java
    DefaultBeanFactory factory = null;
    XmlBeanDefinitionReader reader = null;

    @Before
    public void setUp(){
        factory = new DefaultBeanFactory();
        reader = new XmlBeanDefinitionReader(factory);
    }

    @Test
    public void testGetBean(){
        
        reader.loadBeanDefinitions("petstore-v1.xml");

        PetStoreService petStore =   (PetStoreService)factory.getBean("petStore");

        assertNotNull(petStore);
    }
```

上面的去除重复的测试用例代码

@Before

每次调用测试用例setUp都会被调用

没运行一个测试用例，reader和factory都会被清空一次。测试用例之间互不影响。



每个测试用例都会调用@Before。factory和reader都是new出来的，各个测试用例之间互不影响。





#### ApplicationContext

在spring中，实际工作中很少会用BeanDefinition。而是使用ApplicationContext，封装了Factory和DefinitionReader。后面从ApplicationContext中获取Bean而不用关心底层的实现和处理。

创建一个新的Test。 ApplicationContextTest

```java
public class ApplicationContextTest {

    @Test
    public void testGetBean(){
        ApplicationContext ctx = new ClassPathXmlApplicationContext("petstore-v1.xml");
        PetStoreService petStoreService = (PetStoreService)ctx.getBean("petStore");
        Assert.assertNotNull(petStoreService);
    }
}
```

ApplicationContext是一个接口。

ClassPathXmlApplicationContext是一个实现，从类路径下寻找文件。

ApplicationContext有一个方法，getBean：从Context中把petStore这个bean取出。但是这个方法已经在BeanFactory中定义过了。因此可以让ApplicationContext去扩展BeanFactory接口，这样ApplicationContext就具有getBean方法了。

![](https://i.loli.net/2018/07/29/5b5d55783c730.jpg)

根据上图，把BeanFactory两个关于BeanDefinition的两个方法挪到BeanDefinitionRegistry中。 

ApplicationContext是给客户使用的，继承了BeanFactory，它的实现就是ClassPathXmlApplicationContext



```
ClassPathXmlApplicationContext
```

```java
public class ClassPathXmlApplicationContext implements ApplicationContext {

    private DefaultBeanFactory factory = null;

    public ClassPathXmlApplicationContext(String conigFile) {
       factory = new DefaultBeanFactory();
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
        reader.loadBeanDefinitions(conigFile);
    }

    public Object getBean(String beanID) {
        return factory.getBean(beanID);
    }
}
```

实现了ApplicationContext内部有了一个DefaultBeanFactory的实例。

然后在构造中，new出来reader和factory。

loadBeanDefinitions过程当中，DefaultBeanFactory会不断的调用BeanDefinition的方法registerBeanDefinition。于是这些Defnition就会进入到factory中。

持有所有的实例，就可以通过getBean来获取

#### filesysytemapplication和ClassPathXmlApplicationContext的共同点----Resource

Rresource类图

![](https://i.loli.net/2018/07/29/5b5d5efbf1eda.png)

都会变成inputstream送给dom4j去解析

ResourceTest

```java
public class ResourceTest {
    @Test
    public void testClassPathResource() throws Exception{
        Resource r = new ClassPathResource("petstore-v1.xml");

        InputStream is = null;

        try {
            is = r.getInputStream();
            Assert.assertNotNull(is);
        }finally {
            if (is != null){
                is.close();
            }
        }
    }

    @Test
    public void testFileSystemResource() throws Exception{
        Resource r = new FileSystemResource("/Users/mac/Documents/workspace/litespring/src/test/resources/petstore-v1.xml");

        InputStream is = null;

        try {
            is = r.getInputStream();
            Assert.assertNotNull(is);
        }finally {
            if (is != null){
                is.close();
            }
        }
    }
}
```

ClassPathResource

```java
public class ClassPathResource implements Resource {

    private String path;
    private ClassLoader classLoader;

    public ClassPathResource(String path) {
        this(path,(ClassLoader)null);
    }

    public ClassPathResource(String path, ClassLoader classLoader) {
        this.path = path;
        this.classLoader =(classLoader !=null? classLoader: ClassUtils.getDefaultClassLoader());
    }

    public InputStream getInputStream() throws IOException {
        InputStream is = this.classLoader.getResourceAsStream(this.path);

        if (is == null){
            throw new FileNotFoundException(path+"cannot be opened");
        }
        return is;
    }

    public String getDescription() {
        return this.path;
    }
}
```

```
FileSystemXmlApplicationContext
```

```java
public class FileSystemXmlApplicationContext implements ApplicationContext {

    private DefaultBeanFactory factory = null;

    public FileSystemXmlApplicationContext(String configFile) {
        factory = new DefaultBeanFactory();
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
        Resource resource = new FileSystemResource(configFile);
        reader.loadBeanDefinitions(resource);
    }

    @Override
    public Object getBean(String beanID) {
        return factory.getBean(beanID);
    }
}
```

这个类与classpath的区别几乎只有Resource的区别。

模版方法

![](https://i.loli.net/2018/07/29/5b5d717a04950.png)

抽象出一个新的抽象类，实现了applicationContext的接口

 ![](http://pcq0v117m.bkt.clouddn.com/loaderClass.png)

这里还是让BeanFactory只保留一个getBean()最小集。把一些不常用的，可配置的相关方法放到一个新的接口当中。

ConfigurableBeanFactory



### Setter注入

新增的PetStoreService

```xml
      <bean id="petStore" class="org.litespring.service.v2.PetStoreService" >
          <property name="accountDao" ref="accoutDao"/>
          <property name="itemDao" ref="itemDao"/>
      </bean>
```

我们之前用BeanDefinition表达了<bean>标签中的id, class.这些property怎么表达？

ref怎么表达？

value怎么表达？

这里必须要做点抽象，或者说用类来代表这些感念。

两个类：

- RuntimeBeanReference 
  - - beanName
    - getBeanName()
- TypedStringValue
  - - value
    - getValue()

#### 整体类图

![](http://pcq0v117m.bkt.clouddn.com/fullClassMap.jpg)

**XmlBeanDefinitionReader**

```java
public class XmlBeanDefinitionReader {
	
	public static final String ID_ATTRIBUTE = "id";	

	public static final String CLASS_ATTRIBUTE = "class";
	
	public static final String SCOPE_ATTRIBUTE = "scope";

	public static final String PROPERTY_ELEMENT = "property";
	
	public static final String REF_ATTRIBUTE = "ref";
	
	public static final String VALUE_ATTRIBUTE = "value";
	
	public static final String NAME_ATTRIBUTE = "name";
	
	BeanDefinitionRegistry registry;
	
	protected final Log logger = LogFactory.getLog(getClass());
	
	public XmlBeanDefinitionReader(BeanDefinitionRegistry registry){
		this.registry = registry;
	}
	public void loadBeanDefinitions(Resource resource){
		InputStream is = null;
		try{			
			is = resource.getInputStream();
			SAXReader reader = new SAXReader();
			Document doc = reader.read(is);
			
			Element root = doc.getRootElement(); //<beans>
			Iterator<Element> iter = root.elementIterator();
			while(iter.hasNext()){
				Element ele = (Element)iter.next();
				String id = ele.attributeValue(ID_ATTRIBUTE);
				String beanClassName = ele.attributeValue(CLASS_ATTRIBUTE);
				BeanDefinition bd = new GenericBeanDefinition(id,beanClassName);
				if (ele.attribute(SCOPE_ATTRIBUTE)!=null) {					
					bd.setScope(ele.attributeValue(SCOPE_ATTRIBUTE));					
				}
				parsePropertyElement(ele,bd); 
				this.registry.registerBeanDefinition(id, bd);
			}
		} catch (Exception e) {		
			throw new BeanDefinitionStoreException("IOException parsing XML document from " + resource.getDescription(),e);
		}finally{
			if(is != null){
				try {
					is.close();
				} catch (IOException e) {					
					e.printStackTrace();
				}
			}
		}
		
	}
	public void parsePropertyElement(Element beanElem, BeanDefinition bd) {
		Iterator iter= beanElem.elementIterator(PROPERTY_ELEMENT);
		while(iter.hasNext()){
			Element propElem = (Element)iter.next();
			String propertyName = propElem.attributeValue(NAME_ATTRIBUTE);
			if (!StringUtils.hasLength(propertyName)) {
				logger.fatal("Tag 'property' must have a 'name' attribute");
				return;
			}
			
			
			Object val = parsePropertyValue(propElem, bd, propertyName);
			PropertyValue pv = new PropertyValue(propertyName, val);
			
			bd.getPropertyValues().add(pv);
		}
		
	}
	
	public Object parsePropertyValue(Element ele, BeanDefinition bd, String propertyName) {
		String elementName = (propertyName != null) ?
						"<property> element for property '" + propertyName + "'" :
						"<constructor-arg> element";

		
		boolean hasRefAttribute = (ele.attribute(REF_ATTRIBUTE)!=null);
		boolean hasValueAttribute = (ele.attribute(VALUE_ATTRIBUTE) !=null);
		
		if (hasRefAttribute) {
			String refName = ele.attributeValue(REF_ATTRIBUTE);
			if (!StringUtils.hasText(refName)) {
				logger.error(elementName + " contains empty 'ref' attribute");
			}
			RuntimeBeanReference ref = new RuntimeBeanReference(refName);			
			return ref;
		}else if (hasValueAttribute) {
			TypedStringValue valueHolder = new TypedStringValue(ele.attributeValue(VALUE_ATTRIBUTE));
			
			return valueHolder;
		}		
		else {
			
			throw new RuntimeException(elementName + " must specify a ref or value");
		}
	}
}
```

实际情况中，我们说需要将ref。new出来的。

将一个名称变为实例，



#### 注入bean

TDD的思路

- 1. 针对接口写“high level”的测试用例，暂时不用pass，可以保持fail状态
     - 例如ApplicationContextTestV2
  2. 对一些子任务写测试用例
     - BeanDefinitionTestV2 ：
     - BeanDefinitionValueResolverTest ：测试ref和value
  3. 把子任务生成的代码，类组合起来，让high level的测试用例pass





```java
	protected void populateBean(BeanDefinition bd, Object bean){
		List<PropertyValue> pvs = bd.getPropertyValues();
		
		if (pvs == null || pvs.isEmpty()) {
			return;
		}
		
		BeanDefinitionValueResolver valueResolver = new BeanDefinitionValueResolver(this);
		SimpleTypeConverter converter = new SimpleTypeConverter(); 
		try{
			
			BeanInfo beanInfo = Introspector.getBeanInfo(bean.getClass());
			PropertyDescriptor[] pds = beanInfo.getPropertyDescriptors();
			
			for (PropertyValue pv : pvs){
				String propertyName = pv.getName();
				Object originalValue = pv.getValue();
				Object resolvedValue = valueResolver.resolveValueIfNecessary(originalValue);			
				
				for (PropertyDescriptor pd : pds) {
					if(pd.getName().equals(propertyName)){
						Object convertedValue = converter.convertIfNecessary(resolvedValue, pd.getPropertyType());
						pd.getWriteMethod().invoke(bean, convertedValue);
						break;
					}
				}
 
				
			}
		}catch(Exception ex){
			throw new BeanCreationException("Failed to obtain BeanInfo for class [" + bd.getBeanClassName() + "]", ex);
		}	
	}
```

这里要实现setter的注入

现在唯一知道的就说它的名称，可以使用javabean使用的功能。

```java
			BeanInfo beanInfo = Introspector.getBeanInfo(bean.getClass());
			PropertyDescriptor[] pds = beanInfo.getPropertyDescriptors();
			for(PropertyDescriptor pd : pds){
                if(pd.getName().equals(proertyName)){
                    pd.getWriteMethod().invoke(bean,resolvedValue);
                    break;
                }
			}
```

代码中的bean就是petStoreService

Introspector可以拿到bean的info也就是拿到petStoreService的相关信息。

然后调用getPropertyDescriptors（）属性的描述器

proertyName就相当于bean的名称。

如果相等就认为找到了这个属性，就调用它的getWriteMethod其实就是它的get和set方法。 

通过反射的方式，bean是对象本身，resolvedValue就是AccountDao

下一个循环就可能是把itemDao也set进去。



此时再运行测试用例

```java
    @Test
    public void testGetBeanProperty(){
        ApplicationContext ctx = new ClassPathXmlApplicationContext("petstore-v2.xml");
        PetStoreService petStore = (PetStoreService)ctx.getBean("petStore");

        assertNotNull(petStore.getAccountDao());
        assertNotNull(petStore.getItemDao());
        
        assertTrue(petStore.getAccountDao() instanceof AccountDao);
        assertTrue(petStore.getItemDao() instanceof ItemDao);
    }
```

不报错就证明已经注入进去了。



那么此时我们测试了是一个ref的属性。

#### 注入value

现在将

```xml
<property name="owner" value="liuxin"/>
```

加入到PetStoreService中以及它的getter和setter方法。

此时再增加一个int类型的



```
<property name="version"  value="2"/>
```

此时再运行测试用例就会失败

我们在xml文件当中没有类型只有字符串。所在setVersion的时候就会报错，因为不是int类型的。

所以一定要根据类型去转换。

目标：做转换

#### 实现类型转换

![](http://pcq0v117m.bkt.clouddn.com/Spring%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2%E8%AE%BE%E8%AE%A1%E5%9B%BE.jpg)	

上图是Spring做类型转换的设计图

创建一个想做类型转换的接口，它只有一个方法。

convertIfNessessary如果有必要的话就进行一个转换，两个参数，一个是2一个是类型

传递给这个方法后，这个方法会返回一个整型的值

它的实现是SimpleTypeConverter利用了java的beans

想实现这个功能，先实现小功能，依次通过测试用例来实现

先写一个测试类

```java
public class CustomNumberEditorTest {

	@Test
	public void testConvertString() {
        //true 代表是否允许为null
		CustomNumberEditor editor = new CustomNumberEditor(Integer.class,true);
		
		editor.setAsText("3");
		Object value = editor.getValue();
		Assert.assertTrue(value instanceof Integer);		
		Assert.assertEquals(3, ((Integer)editor.getValue()).intValue());
		
		
		editor.setAsText("");
		Assert.assertTrue(editor.getValue() == null);
		
		
		try{
			editor.setAsText("3.1");
			
		}catch(IllegalArgumentException e){
			return ;
		}
		Assert.fail();		
		
	}

}
```



Setter注入的回顾

- 1.引入新的概念PropertyValue
  - RuntimeBeanReference
  - TypedStringValue
- 2.用BeanDefinitionResolver去resolve
- 3.使用TypeConverter去把字符的值进行转型



#### 课后答疑

我们用BeanDefinition表达<bean>标签中的id，class

那么property ref value怎么表达

 

ref如恶化保证从factory中获取的时候已经是一个bean

 

### 使用构造器注入

#### 解析xml

**xml文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
	
  <bean id="petStore"
        class="org.litespring.service.v3.PetStoreService" >  
   	<constructor-arg  ref="accountDao"/>
   	<constructor-arg  ref="itemDao"/>
   	<constructor-arg  value="1"/>       
  </bean>

   <bean id="itemDao" class="org.litespring.dao.v3.ItemDao">
   
  </bean>
  <bean id="accountDao"  class="org.litespring.dao.v3.AccountDao">
   
  </bean>

</beans> 
```

在实现层面肯定要去解析xml才可以。假设，在解析它们的时候，解析完需要用一个类来表达这些构造器的参数。

所以仍然要做一个设计，请先查看类图。

![](http://pcq0v117m.bkt.clouddn.com/Contstructor-arg.png)

会新增加一个类ConstructorArgument

里面会有一个list，list里面的ValueHolder是ref或value

那么为什么不能将List<ValueHolder>中的 ValueHolder换成Object呢？

其实Spring中支持的Constructor-arg是比较复杂的

![](http://pcq0v117m.bkt.clouddn.com/Spring%E6%94%AF%E6%8C%81%E7%9A%84constructor%E7%B1%BB%E5%9E%8B.png)

type指定类型

name表示注入到参数名称是years中，指定参数名称。

index指定参数顺序。

因此我们这里简化了，这里只用value

在ConstructorArgument中有一个静态的

```
public static class ValueHolder
```

静态内部类，因为它只在ConstructorArgument中使用，只用来描述一个构造函数的参数，高内聚。

这里我们只用了value

#### 和setter做对比

- 和setter注入做对比
  - 设计一个数据结构PropertyValue/ConstructorArgument
  - 解析XML，填充这个数据结构
  - 利用这个数据结构做事情

#### 实现构造器注入

如果一个类有多个构造函数，那么怎么选取

![](http://pcq0v117m.bkt.clouddn.com/%E9%80%89%E5%8F%96%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0.png)

找到参数合适的之后也要匹配参数对象类型是否一致，能不能对这个类型做赋值或转型操作。



设计一个类来解决这个问题。

![](http://pcq0v117m.bkt.clouddn.com/%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2Construstor.png)

可以把一个ref变成一个实际的类型对象出来。

通过find来找到这个对象，然后通过autowire来创建对象。在创建对象的过程当中就会把三个参数的值注入到对象中去了。这个就是ConstructorResolver的作用。

BeanDefinition可以获取ConstructorArgument

ConstructorResolver

```java
ublic class ConstructorResolver {

	protected final Log logger = LogFactory.getLog(getClass());
	
	
	private final ConfigurableBeanFactory beanFactory;


	
	public ConstructorResolver(ConfigurableBeanFactory beanFactory) {
		this.beanFactory = beanFactory;
	}

	
	public Object autowireConstructor(final BeanDefinition bd) {
		
		Constructor<?> constructorToUse = null;		//找到一个可以用的Constructor
		Object[] argsToUse = null;	//PetStoreService需要有参数，这里用Objcet来记录
	
		Class<?> beanClass = null; //通过BeanFactory来获取，每次进来都装载
		try {
			beanClass = this.beanFactory.getBeanClassLoader().loadClass(bd.getBeanClassName());
			
		} catch (ClassNotFoundException e) {
			throw new BeanCreationException( bd.getID(), "Instantiation of bean failed, can't resolve class", e);
		}	
		
		
		Constructor<?>[] candidates = beanClass.getConstructors();	//反射方法
		
		
		BeanDefinitionValueResolver valueResolver =
				new BeanDefinitionValueResolver(this.beanFactory);
		
		ConstructorArgument cargs = bd.getConstructorArgument();
        
		SimpleTypeConverter typeConverter = new SimpleTypeConverter();
		
		for(int i=0; i<candidates.length;i++){
			
			Class<?> [] parameterTypes = candidates[i].getParameterTypes();//拿到两个参数
			if(parameterTypes.length != cargs.getArgumentCount()){//是否和xml中的数量相等
				continue; //不管用，找第二个构造函数的循环
			}			
			argsToUse = new Object[parameterTypes.length]; 
			
			boolean result = this.valuesMatchTypes(parameterTypes,  //值是否和ref匹配
					cargs.getArgumentValues(), 
					argsToUse, 
					valueResolver, 
					typeConverter);
			
			if(result){
				constructorToUse = candidates[i];//如果匹配成功，这个候选的构造函数就是我们要的
				break;
			}
			
		}
		
		
		//找不到一个合适的构造函数
		if(constructorToUse == null){
			throw new BeanCreationException( bd.getID(), "can't find a apporiate constructor");
		}
		
		
		try {
			return constructorToUse.newInstance(argsToUse); //传递对象
		} catch (Exception e) {
			throw new BeanCreationException( bd.getID(), "can't find a create instance using "+constructorToUse);
		}		
		
		
	}
	
	private boolean valuesMatchTypes(Class<?> [] parameterTypes,
			List<ConstructorArgument.ValueHolder> valueHolders,
			Object[] argsToUse,
			BeanDefinitionValueResolver valueResolver,
			SimpleTypeConverter typeConverter ){
		
		
		for(int i=0;i<parameterTypes.length;i++){
			ConstructorArgument.ValueHolder valueHolder 
				= valueHolders.get(i);
			//获取参数的值，可能是TypedStringValue, 也可能是RuntimeBeanReference
			Object originalValue = valueHolder.getValue();
			
			try{
				//获得真正的值
				Object resolvedValue = valueResolver.resolveValueIfNecessary( originalValue);
				//如果参数类型是 int, 但是值是字符串,例如"3",还需要转型
				//如果转型失败，则抛出异常。说明这个构造函数不可用
				Object convertedValue = typeConverter.convertIfNecessary(resolvedValue, parameterTypes[i]);
				//转型成功，记录下来
				argsToUse[i] = convertedValue;
			}catch(Exception e){
				logger.error(e);
				return false;
			}				
		}
		return true;
	}
	

}
```







### Commons-BeanUtils





