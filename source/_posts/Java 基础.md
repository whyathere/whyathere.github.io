---
title: Java 基础
date: 2018-06-05 10:03:30
tags: Java 基础
categories: Java 基础

---

### 泛型

**在实例化类的时候指明泛型的具体类型**

**`类型的参数化，可以把类型像方法的参数那样传递。`**

优点：**`泛型使编译器可以在编译期间对类型进行检查以提高类型安全，减少运行时由于对象类型不匹配引发的异常。如果我们在对一个对象赋值不符合其泛型的规范，就会编译报错`**

1、代码重用：我们可以编写一个方法/类/接口，并用于我们想要的任何类型。

2、类型安全：泛型使错误显示在编译时而不是在运行时

例如：

```java
// A Simple Java program to demonstrate that NOT using 
// generics can cause run time exceptions 
import java.util.*; 
  
class Test 
{ 
    public static void main(String[] args) 
    { 
        // Creatinga an ArrayList without any type specified 
        ArrayList al = new ArrayList(); 
  
        al.add("Sachin"); 
        al.add("Rahul"); 
        al.add(10); // Compiler allows this 
  
        String s1 = (String)al.get(0); 
        String s2 = (String)al.get(1); 
  
        // Causes Runtime Exception 
        String s3 = (String)al.get(2); 
    } 
} 
```

这段代码只有在运行时才会报错。

```java
Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
```

**类型转换异常**

**泛型如何解决这个问题？**

在定义ArrayList时，我们可以指定此列表只能使用String对象。

```java
import java.util.*; 
  
class Test 
{ 
    public static void main(String[] args) 
    { 
        // Creating a an ArrayList with String specified 
        ArrayList <String> al = new ArrayList<String> (); 
  
        al.add("Sachin"); 
        al.add("Rahul"); 
  
        // Now Compiler doesn't allow this 
        al.add(10);  
  
        String s1 = (String)al.get(0); 
        String s2 = (String)al.get(1); 
        String s3 = (String)al.get(2); 
    } 
} 
```

3、不需要单独类型转换：如果我们不使用泛型，那么，在上面的例子中，每次我们从ArrayList检索数据时，我们必须对它进行强制类型转换，而且还要自己检查是否是String类型，否则就会报错，因为从List中取出元素时，其类型会默认为Object。在每次检索操作中进行类型转换都是一个令人头痛的问 如果我们已经知道我们的列表只包含字符串数据，那么我们不需要每次都对它进行类型转换。

4、实现通用算法：通过使用泛型，我们可以实现适用于不同类型对象的算法，同时它们也是类型安全的。

5、有界泛型

***在使用泛型时，我们会有这种需求：需要指定泛型的类型范围。有界类型就是在类型参数部分指定extends或super关键字，这里的extends也含有implements的功能，分别用上限或下限来限制类型范围，从而限制泛型的类型边界。例如：***

```java
<T extends Animal>//限定T是Animal的子类

<T super Dog >//限定T是Dog的超类
```

```java
public void killAll(ArrayList<T extends Animal> animals){...};
```



***多个限定时我们可以使用&来进行分割，这时关键词只能使用extends。与多重继承类似，这里只可以有一个类，其他都是接口。***

```java
<T extends Object&Comparable&Serializable>
```

#### 泛型方法

**在调用方法的时候指明泛型的具体类型**

`定义泛型方法语法格式如下`

![](https://user-images.githubusercontent.com/24977343/58958544-ba9ac600-87d5-11e9-9473-ea75ecb5077f.png)

`调用泛型方法语法格式如下`

![](https://user-images.githubusercontent.com/24977343/58958730-25e49800-87d6-11e9-9619-7774468f4e20.png)

​       **说明一下，定义泛型方法时，必须在返回值前边加一个<T>，来声明这是一个泛型方法，持有一个泛型T，然后才可以用泛型T作为方法的返回值。**

​       Class<T>的作用就是指明泛型的具体类型，而Class<T>类型的变量c，可以用来创建泛型类的对象。

​       为什么要用变量c来创建对象呢？既然是泛型方法，就代表着我们不知道具体的类型是什么，也不知道构造方法如何，因此没有办法去new一个对象，但可以利用变量c的newInstance方法去创建对象，也就是利用反射创建对象。

​       泛型方法要求的参数是Class<T>类型，而Class.forName()方法的返回值也是Class<T>，因此可以用Class.forName()作为参数。其中，forName()方法中的参数是何种类型，返回的Class<T>就是何种类型。在本例中，forName()方法中传入的是User类的完整路径，因此返回的是Class<User>类型的对象，因此调用泛型方法时，变量c的类型就是Class<User>，因此泛型方法中的泛型T就被指明为User，因此变量obj的类型为User。

​       当然，泛型方法不是仅仅可以有一个参数Class<T>，可以根据需要添加其他参数。

​       为什么要使用泛型方法呢？因为泛型类要在实例化的时候就指明类型，如果想换一种类型，不得不重新new一次，可能不够灵活；而泛型方法可以在调用的时候指明类型，更加灵活。



## Java数据结构应用

### List

#### subList()方法

截取list的一段，但是如果对这一段做了修改，那么原来的list也会发生相应的变化。

## 抽象类

**定义：**在面向对象的概念中，所有的对象都是通过类来描绘的，但是反过来，并不是所有的类都是用来描绘对象的，如果一个类中没有包含足够的信息来描绘一个具体的对象，这样的类就是抽象类。

**简单定义：**有一个或多个抽象方法的类，必须声明为抽象类。不能创建实例

抽象类除了不能实例化对象之外，类的其它功能依然存在，成员变量、成员方法和构造方法的访问方式和普通类一样。

由于抽象类不能实例化对象，所以抽象类必须被继承，才能被使用。也是因为这个原因，通常在设计阶段决定要不要设计抽象类。

父类包含了子类集合的常见的方法，但是由于父类本身是抽象的，所以不能使用这些方法。

在Java中抽象类表示的是一种继承关系，一个类只能继承一个抽象类，而一个类却可以实现多个接口。



### 为啥用抽象类

1. 父类中的方法确实没有必要写(通用方法)，当然可以用类的覆盖，但是没有必要
2. 各个子类中的这个方法肯定会不同
3. 抽象方法有提示作用
4. 发现共性，向上抽取。方法功能声明相同，但是方法功能主体不同。

总结：

1. 抽象类不能被实例化
2. 抽象类中不一定包含抽象方法，但是有抽象方法的类必定是抽象类
3. 抽象类中的抽象方法只是声明，不包含方法体，就是不给出方法的具体实现也就是方法的具体功能。
4. 构造方法，类方法(用static修饰的方法)不能声明为抽象方法
5. 抽象类的子类必须给出抽象类中的抽象方法的具体实现，除非该子类也是抽象类
6. 具体派生类必须覆盖基类的抽象方法
7. 抽象派生类可以覆盖基类的抽象方法，也可以不覆盖。如果不覆盖，则其具体派生类必须覆盖它们

