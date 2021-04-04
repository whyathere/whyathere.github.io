---
title: Lambda表达式
date: 2018-09-10 18:27:21
tags: Lambda		
---

## Lambda表达式

### 简介

Lambda表达式可以替代只有一个抽象函数的接口实现，告别匿名内部类，代码看起来更简洁易懂。Lambda表达式同时还提升了对集合、框架的迭代、遍历、过滤数据的操作。

### 特点

- 函数式编程
- 参数类型自动推断
- 代码量少、简洁

### 应用场景

任何有函数式接口的地方

### 函数式接口

Supplier代表一个输出

Consumer代表一个输入

BiConsumer代表两个输入



Function代表一个输入，一个输出(一般输入和输出是不同类型的)

UnaryOperator代表一个输入，一个输入(输入和输出是相同类型的)



BiFunction代表两个输入，一个输出(一般输入和输出是不同类型的)

BinaryOperator代表两个输入，一个输出(输入和输出是相同类型的)





```java
 public static void main(String[] args) throws Exception {
        TeacherDao teacherDao = new TeacherDao() {
            @Override
            public int get(Teacher teacher) {
                return 1;
            }
        };
        System.out.println(teacherDao.get(new Teacher()));

        TeacherDao td1 = (teacher)->{return 2;};
        TeacherDao td2 = (Teacher teacher)->{return 3;};
        TeacherDao td3 = (teacher)->4;
        TeacherDao td4 = (Teacher teacher)->5;
        System.out.println(td1.get(new Teacher()));
        System.out.println(td2.get(new Teacher()));
        System.out.println(td3.get(new Teacher()));
        System.out.println(td4.get(new Teacher()));

        //Function必须有一个输入str一个输出Integer(str.length())
        Function<String,Integer> f1 = (str)->{return str.length();};
        Integer sdfasdfdads = f1.apply("sdfasdfdads");
        System.out.println(sdfasdfdads);
        //只有一个输出
        Supplier<String> supplier = ()->{return "sdfasf";};
        Supplier<String> supplier1 = ()->"sdf1212asf";
        Supplier<String> supplier2 = ()->StreamDemo.gen1();
        Supplier<String> supplier3 = StreamDemo::gen1;


        //只有一个输入
        Consumer<String> consumer = (str)-> System.out.println(str);
        consumer.accept("sdafasf");

        //两个输入一个输出，输入和输出是不同类型的
        BiFunction<String,String,Integer> function = (str1,str2)->{return str1.length()+str2.length();};
        Integer apply = function.apply("123", "456");
        System.out.println(apply);

        //两个输入一个输出，输入和输出是相同类型的
        BinaryOperator<String>  binaryOperator = (n1,n2)->n1+n2;
        String apply1 = binaryOperator.apply("121212", "1231232131");
        System.out.println(apply1);

        List<String> list = Arrays.asList("a","b","c");
        list.forEach(System.out::println);

    }
```

- Function<String,Integer> f1 = (str)->{return str.length();};

小括号里的是参数，后面的是返回。

- Supplier<String> supplier3 = StreamDemo::gen1;

::相当于调用这个方法

### 方法引用的分类

|     类型     |        语法        |         对应的lambda表达式         |
| :----------: | :----------------: | :--------------------------------: |
| 静态方法引用 | 类名::staticMethod |  (args)->类名.staticMethod(args)   |
| 实例方法引用 |  Inst::instMethod  |   (args)->inst.instMethod(args)    |
| 对象方法引用 |  类名::instMethod  | (inst.args)->类名.instMethod(args) |
| 构造方法引用 |     类名::new      |       (args)->new 类名(args)       |



### Stream

#### 简介

Stream是一组用来处理数组、集合的API，例如过滤、映射、规约、排序、查询记录

- Java8之所以引入函数式编程原因有：
  - 代码简洁函数式编程写出的代码简洁且意图明确，使用stream接口让你从此告别for循环
  - 多核友好，Java函数式编程使得编写并行程序从未如此简单，你需要的全部就是调用一下parallel()方法

#### 特性

1. 不是数据结构，没有内部存储
2. 不支持索引搜索
3. 延迟计算
4. 支持并行
5. 很容易生成数据或集合(List、Set)
6. 支持过滤、查找、转换、汇总、聚合等操作

#### Stream运行机制

Stream分为源source、中间操作、终止操作

流的源可以是一个数组、一个集合、一个生成器方法、一个I/O通道等等。

一个流可以有零个和或者多个中间操作，每一个中间操作都会返回一个新的流，供下一个操作使用。一个流只会有一个终止操作。

Stream只会遇到终止操作，它的源才开始执行遍历操作。

#### 常用API

终止操作：

- 循环：forEach
- 计算：min、max、count、average
- 匹配：anyMatch、allMatch、noneMatch、findFirst、findAny
- 汇聚：reduce
- 收集器：toArray collect

#### 例子

##### 数组

```java
    //通过数组来生成
    static void gen1(){
        String[] strs={"a","b","c","d"};
        Stream<String> strs1 = Stream.of(strs);
        strs1.forEach(System.out::println);
        Arrays.stream(strs).forEach(System.out::println);
    }
```

##### 集合

```java
//集合
static void gen2(){
    List<String> list = Arrays.asList("1","2","3","4");
    list.stream().forEach(System.out::println);
}
```

##### 过滤

```java
        String[] strs={"a","b","c","d"};
        Arrays.stream(strs).filter((x)->!x.equals("c")).forEach(System.out::println);
```

##### 元素个数

```java
long c = Arrays.stream(strs).filter((x) -> !x.equals("c")).count();
System.out.println(c);
```

##### 转换求和

```java
int sum = Arrays.asList(1, 2, 3, 4, 5).stream().mapToInt(x -> x).sum();
System.out.println(sum);
```

##### 求集合最大值

```java
OptionalInt max = Arrays.asList(1, 2, 3, 4, 5).stream().mapToInt(x -> x).max();
System.out.println(max.getAsInt());
```

##### 按照要求取值

```java
Optional<Integer> any = Arrays.asList(1, 2, 3, 4, 5).stream().filter(x -> x % 2 == 0).findAny();
Optional<Integer> any1 = Arrays.asList(1, 2, 3, 4, 5).stream().filter(x -> x % 2 == 0).findFirst();
```

##### 排序

```java
String[] strs={"b","c","d","a",};
Arrays.stream(strs).sorted().forEach(System.out::println);
```

##### 获取最大最小值但是不使用min和max方法

```java
Optional<Integer> first = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8).stream().sorted().findFirst();
System.out.println(first.get());
```

```java
Optional<Integer> first = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8).stream().sorted((a, b) -> b - a).findFirst();
System.out.println(first.get());
```

##### 最终返回一个集合

```java
List<Integer> collect = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8).stream().collect(Collectors.toList());
System.out.println(collect);
```

##### 去重

```java
Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8).stream().distinct();
```

```java
Arrays.asList(1, 2, 2, 4, 5, 6, 7, 8).stream().collect(Collectors.toSet()).forEach(System.out::println);
```

##### 打印20-30的集合数据

```java
Stream.iterate(1,x->x+1).limit(50).skip(20).limit(10);
```

```java
Stream.of(str.split(",")).mapToInt(x -> Integer.valueOf(x)).sum()
```

```java
Stream.of(str.split(",")).mapToInt(Integer::valueOf).sum();
```

##### 求和的过程中将每个数值都打印出来，同时算出最终的求和结果

```java
String str = "11,22,33,44";
System.out.println(Stream.of(str.split(",")).peek(System.out::println).mapToInt(Integer::valueOf).sum());
```

# 自定义注解

## 特点

- Anontation是Java5开始引入的新特征，中文名叫注解。
- 它提供了一种安全的类似注释的机制，用来将任何的信息或元数据(metadata)与程序元素(类、方法、成员变量等)进行关联。
- 为程序的元素(类、方法、成员变量)加上更直观更明了的说明，这些说明信息是与程序的业务逻辑无关，并且提供指定的工具或框架使用。
- Anontation像一种修饰符一样，应用于包、类型、构造方法、方法、成员变量、参数及本地变量的声明语句中。
- Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。
- 注解不会也不影响代码的实际逻辑，仅仅起到辅助性的作用。包含在java.lang.annotation包中。

## 注解的作用

- 生成文档。
- 跟踪代码依赖性，实现替代配置文件功能。
- 在编译时进行格式检查。如@override放在方法前，如果你这个方法并不是覆盖了超类方法，则编译时就能检查出。

## 元注解

- 元注解的作用是负责注解其他注解，java中定义了四个标准的meta-annotation类型，他们被用来提供对其他annotation类型作说明
- 这些类型和它们所支持的类在java.lang.annotation包中
  - @Target：用来描述注解的使用范围(注解可以用在什么地方)
  - @Retention：表示需要在什么级别保存该注解信息，描述注解的生命周期
    - Source<Class<Runtime
  - @Document：说明该注解将被包含在javadoc中
  - @Inherited：说明子类可以继承父类中的该注解



## 例子

### 过时

```java
@Deprecated
public int getAge(){
    return 1;
}
```

```java
@Myannotation(name = "lisi",age = 16)
public class Meta1Annotation{
    
}

//用来声明当前自定义的注解适合用于什么地方，类、方法、变量、包等
@Target({ElementType.METHOD,ElementType.TYPE})
//用来表示当前注解适用于什么环境，是源码级别还是类级别还是运行环境，一般都是运行时环境
@Retention(RetentionPolicy.RUNTIME)
//是否显示在javadoc中
@Documented
//表示当前注解能否被继承
@Inherited
@interface Myannotation{
//定义的方式看起来像是方法，但是实际上使用注解的时候填写的参数的名称,一般默认的名称是value
    //自定义注解中填写的所有方法都需要在使用注解的时候，添加值，很麻烦，因此包含默认值
    String name() default "zhangsan";
    int age() default 12;
}
```

## 原理

自定义注解的原理是反射

