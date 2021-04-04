### 线程同步

#### synchronized

- 锁的是对象不是代码
- this XX.class
- 锁定方法 非锁定方法  同时执行
- 锁升级
  - 偏向锁  自旋锁 重量级锁

锁升级：synchronized内部进行了一些优化，比如要锁定某个对象，在对象头上记录什么类型的锁，那个线程申请了这个锁。

- 偏向锁：线程来了先不加锁 ，只是记录id值，认为线程独有。下次再来还是第一次的线程独有，第二次只判断是否是之前的，如果是之前的就不加锁，直接执行。如果id不等，就进行锁升级；首先进行升级，偏向锁升级为自旋锁，另外一个线程进行自旋，默认值为10(转10圈)。如果还是拿不到，就升级为重量级锁，经过OS进入等待队列，就不在占用CPU时间了(占用CPU10圈的时间)。进入等待队列

Lock很多用的CAS操作，占用CPU时间。

synchronized最终不占用CPU时间。

**什么时候用自旋锁什么时候用重量级锁**

自旋锁：线程数少的时候(积极的排队，占用CPU时间)，耗时短

重量级锁：线程数多的时候(wait 不占用CPU时间)，操作消耗时间长的



#### volatile

本身含义：可变的、易变的、容易产生改变的。可以保证可见性和重排序，不能保证原子性。synchronized原子性

**作用**

- 保证线程可见性
  - MESI
  - 缓存一致性协议
- 禁止指令重排序
  - DCL单例
  - Double Check Lock
  - Mgr06.java

堆内存：所有线程共享，每个线程都有自己的工作内存。

#### 单例模式

**饿汉式**

1. 类的构造方法设置为private,这样就只有自己能new。(get instance)
2. 没有用的时候就初始化了

```java
public class Mgr01 {

    private static final Mgr01 INSTANCE = new Mgr01();

    private Mgr01(){};

    public static Mgr01 getInstance(){
        return INSTANCE;
    }

    public void m(){
        System.out.println("m");
    }

    public static void main(String[] args) {
        Mgr01 m1 = Mgr01.getInstance();
        Mgr01 m2 = Mgr01.getInstance();
        System.out.println(m1 == m2);
    }
}
```

**懒汉式**



单例模式要双重检查、单例需要加volatile。虽然很不容易出问题。

new对象：申请内存、变量初始化、变量赋值给INSTANCE

#### CAS

自旋锁

atomic开头：用CAS保障线程安全



#### 关键词

read-modify-wtrite更新操作：指对共享变量的更新不是一个简单的赋值操作而是变量的新值依赖于变量的旧值。

volatiel无法保障自增操作的原子性

### 线程池

一.	newCachedThreadPool(无指定线程池大小，来了就创建)

创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们，并在需要时使用提供的ThreadFactory创建新线程特征：

- 线程池中数量没有固定，可达到最大值(Integer.MAX.VALUE)
- 线程池中的线程可进行缓存重复利用和回收(回收默认时间为1分钟)
- 当线程池中，没有可用线程，会重新创建一个线程

二     newFixedThreadPool(制定线程池大小，复用)

三	newSingleThreadExecutor(只有一个线程)







