
---
title: 设计模式
---

<!-- TOC -->

- [常用的几种设计模式](#常用的几种设计模式)
    - [Proxy 代理模式](#proxy-代理模式)
    - [Factory 工厂模式](#factory-工厂模式)
    - [Singleton 单例模式](#singleton-单例模式)
        - [特点](#特点)
        - [一.  饿汉式](#一--饿汉式)
        - [二.  懒汉式](#二--懒汉式)
            - [1. 一般版](#1-一般版)
            - [2. 双重校验锁](#2-双重校验锁)
            - [3. 静态内部类(推荐版本)](#3-静态内部类推荐版本)
            - [4. 防止反射调用](#4-防止反射调用)
            - [5. 序列化一致](#5-序列化一致)
            - [6. 枚举实现](#6-枚举实现)
                - [枚举特性](#枚举特性)
        - [静态类](#静态类)
            - [对比静态类和单例](#对比静态类和单例)
            - [运用场景](#运用场景)
        - [小结：](#小结)
    - [Delegate 委派模式](#delegate-委派模式)
    - [Strategy 策略模式](#strategy-策略模式)
    - [Prototype 原型模式](#prototype-原型模式)
    - [Template 模板模式](#template-模板模式)

<!-- /TOC -->

# 常用的几种设计模式

[Markdown——入门指南](https://www.jianshu.com/p/1e402922ee32/)

## Proxy 代理模式
## Factory 工厂模式
## Singleton 单例模式

### 特点
1、只能有一个实例;<br>
2、单例类必须创建自己的唯一实例;<br>
3、单例类必须向其他对象提供这一实例

>1.为什么静态内部类的单例模式是最推荐的？

>2.如何在反射的情况下保证单例？

>3.如何在反序列化中保证单例？

>4.为什么不适用静态类

### 一.  饿汉式
>优点就是线程安全啦，缺点很明显，类加载的时候就实例化对象了，浪费空间。

```
package com.cp.chengradle.instance;
public class ChenSingleton1 {
    private static ChenSingleton1 instance = new ChenSingleton1();
    private ChenSingleton1 (){}
    public static ChenSingleton1 getInstance() {
        return instance;
    }
}
```

### 二.  懒汉式
#### 1. 一般版
>将synchronized加在方法上性能大打折扣（syncrhonized会造成线程阻塞）

```
public class ChenLazySingleton1 {
    private static ChenLazySingleton1 instance;
    private ChenLazySingleton1 (){}
    public static synchronized ChenLazySingleton1 getInstance() {
        if (instance == null) {
            instance = new ChenLazySingleton1();
        }
        return instance;
    }
```
#### 2. 双重校验锁
>保证了线程安全，又提高了性能。

```
public class LazySingleton1 {
    private static LazySingleton1 instance;
    private LazySingleton1 (){}
    public static LazySingleton1 getInstance() {
        if (instance == null) {  
            synchronized (LazySingleton1.class) {  
            if (instance == null) {  
                instance = new LazySingleton1();
                }  
            }  
        } 
        return instance;
    }
```
#### 3. 静态内部类(推荐版本)
>双重校验锁版，不管性能再如何优越，还是使用了synchronized修饰符，既然使用了该修饰符，那么对性能多多少少都会造成一些影响

一个例子分析静态内部类
```
public class OuterTest {
    static {
        System.out.println("load outer class...");
    }
    // 静态内部类
    static class StaticInnerTest {
        static {
            System.out.println("load static inner class...");
        }
        static void staticInnerMethod() {
            System.out.println("static inner method...");
        }
    }
    public static void main(String[] args) {
        OuterTest outerTest = new OuterTest(); // 此刻其内部类是否也会被加载？
        System.out.println("===========分割线===========");
        OuterTest.StaticInnerTest.staticInnerMethod(); // 调用内部类的静态方法
    }
}
```
结果：
```
load outer class...
===========分割线===========
load static inner class...
static inner method
```

明白两点：

**1. 加载一个类时，其内部类不会同时被加载。**

**2. 一个类被加载，当且仅当其某个静态成员（静态域、构造器、静态方法等）被调用时发生。**

```
public class LazySingleton2 {
    private LazySingleton2() {
    }
    static class SingletonHolder {
        private static final LazySingleton2 instance = new LazySingleton2();
    }
    public static LazySingleton2 getInstance() {
        return SingletonHolder.instance;
    }
}
```
由于对象实例化是在内部类加载的时候构建的，因此该版是线程安全的(因为在方法中创建对象，才存在并发问题，静态内部类随着方法调用而被加载，只加载一次，不存在并发问题，所以是线程安全的)。

另外，在getInstance()方法中没有使用synchronized关键字，因此没有造成多余的性能损耗。当LazySingleton2类加载的时候，其静态内部类SingletonHolder并没有被加载，因此instance对象并没有构建。

而我们在调用LazySingleton2.getInstance()方法时，内部类SingletonHolder被加载，此时单例对象才被构建。因此，这种写法节约空间，达到懒加载的目的，该版也是众多博客中的推荐版本。

#### 4. 防止反射调用

```
public class LazySingleton3 {
    private static boolean initialized = false;
    private LazySingleton3() {
        synchronized (LazySingleton3.class) {
            if (initialized == false) {
                initialized = !initialized;
            } else {
                throw new RuntimeException("单例已被破坏");
            }
        }
    }
    static class SingletonHolder {
        private static final LazySingleton3 instance = new LazySingleton3();
    }
    public static LazySingleton3 getInstance() {
        return SingletonHolder.instance;
    }
}
```
#### 5. 序列化一致

readResolve()方法何时如何调用
ObjectInputStream.readObject()-->ObjectInputStream.readObject0-->ObjectInputStream.readOrdinaryObject-->构造ObjectStreamClass类，判断readResolve()方法是否存在，反射调用readResolve()

```
public class LazyInnerSingleton implements Serializable{
    //改变量用于放置反射创建不同的单例对象
    private static boolean initialized = false;
    private LazyInnerSingleton() {
        synchronized (LazyInnerSingleton.class) {
            if (initialized == false) {
                initialized = !initialized;
            } else {
                throw new RuntimeException("单例已被破坏");
            }
        }
    }
    static class SingletonHolder {
        private static final LazyInnerSingleton instance = new LazyInnerSingleton();
    }
    public static LazyInnerSingleton getInstance() {
        return SingletonHolder.instance;
    }

    //readResolve(）代替了从流中读取对象。这就确保了在序列化和反序列化的过程中没人可以创建新的实例。
    private Object readResolve() {
        return getInstance();
    }
}
```

#### 6. 枚举实现

这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象，可谓是很坚强的壁垒。

单例的枚举实现在《Effective Java》中有提到，因为其**功能完整、使用简洁、无偿地提供了序列化机制、在面对复杂的序列化或者反射攻击时仍然可以绝对防止多次实例化**等优点，单元素的枚举类型被作者认为是实现Singleton的最佳方法。

```
public enum EnumSingleton {  
    INSTANCE;  
    public void getInstance() {  

    }  

}
```
##### 枚举特性
**自由序列化，线程安全，保证单例**

1.由于enum是通过继承了Enum类实现的，enum结构不能够作为子类继承其他类，但是可以用来实现接口。此外，enum类也不能够被继承，在反编译中，我们会发现该类是final的。

2.enum有且仅有private的构造器，防止外部的额外构造，这恰好和单例模式吻合。

3.对于序列化和反序列化，因为每一个枚举类型和枚举变量在JVM中都是唯一的，即Java在序列化和反序列化枚举时做了特殊的规定，枚举的writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法是被编译器禁用的，因此也不存在实现序列化接口后调用readObject会破坏单例的问题。在序列化的时候Java仅仅是将枚举对象的name属性输出到结果中，**反序列化的时候则是通过java.lang.Enum的valueOf方法来根据名字查找枚举对象**。同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法。

```
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                                String name) {
        if (enumType == null)
            throw new NullPointerException("enumType == null");
        if (name == null)
            throw new NullPointerException("Name is null");
        T[] values = getSharedConstants(enumType);
        T result = null;
        if (values != null) {
            for (T value : values) {
                if (name.equals(value.name())) {
                    result = value;
                }
            }
        } else {
            throw new IllegalArgumentException(enumType.toString() + " is not an enum type.");
        }

        if (result != null)
            return result;
        throw new IllegalArgumentException(
            "No enum constant " + enumType.getCanonicalName() + "." + name);
    }
```

4.枚举不能反射调用,看下Constructor类的newInstance方法源码,throw new IllegalArgumentException("Cannot reflectively create enum objects")
```
public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, null, modifiers);
            }
        }
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }
```

```
public enum ChenEnum {
    INSTANCE1(9),
    INSTANCE2(10);
    int age;
    public void printChen() {

    }

    ChenEnum() {
        this.age = 0;
    }

    ChenEnum(int age) {
        this.age = age;
    }

    public static void chen2() {

    }
}
//反编译之后,没有无参构造函数

.class public final enum ChenEnum
.super Enum

public enum ChenEnum {
    public static final enum ChenEnum INSTANCE1;
    public static final enum ChenEnum INSTANCE2;
    int age;

    static {
        ChenEnum.INSTANCE1 = new ChenEnum("INSTANCE1", 0, 9);
        ChenEnum.INSTANCE2 = new ChenEnum("INSTANCE2", 1, 10);
        ChenEnum.$VALUES = new ChenEnum[]{ChenEnum.INSTANCE1, ChenEnum.INSTANCE2};
    }

    private ChenEnum(String arg1, int arg2, int arg3) {
        super(arg1, arg2);
        this.age = arg3;
    }

    private ChenEnum(String arg2, int arg3) {
        super(arg2, arg3);
        this.age = 0;
    }

    public static void chen2() {
    }

    public void printChen() {
    }

    public static ChenEnum valueOf(String arg1) {
        return Enum.valueOf(ChenEnum.class, arg1);
    }

    public static ChenEnum[] values() {
        // Method was not decompiled
    }
}

```
在了解了Java如何处理枚举的定义以及序列化和反序列化枚举类型之后，我们就需要在系统或者类库升级时，对其中定义的枚举类型多加注意，为了保持代码上的兼容性，如果我们定义的枚举类型有可能会被序列化保存(放到文件中、保存到数据库中，进入分布式内存缓存中)，那么我们是不能够删除原来枚举类型中定义的任何枚举对象的，否则程序在运行过程中，JVM就会抱怨找不到与某个名字对应的枚举对象了。另外，在远程方法调用过程中，如果我们发布的客户端接口返回值中使用了枚举类型，那么服务端在升级过程中就需要特别注意。如果在接口的返回结果的枚举类型中添加了新的枚举值，那就会导致仍然在使用老的客户端的那些应用出现调用失败的情况。因此，针对以上两种情况，**应该尽量避免使用枚举序列化，如果实在要用，也需要仔细设计，因为一旦用了枚举，有可能会给后期维护带来隐患**。

### 静态类
误解一：静态方法常驻内存而实例方法不是。

实际上，特殊编写的实例方法可以常驻内存，而静态方法需要不断初始化和释放。

误解二：静态方法在堆(heap)上，实例方法在栈(stack)上。

实际上，都是加载到特殊的不可写的代码内存区域中。

#### 对比静态类和单例
1. 单例可以继承和被继承，方法可以被override，而静态方法不可以。

2. 静态方法中产生的对象会在执行后被释放，进而被GC清理，不会一直存在于内存中。

3. 静态类会在第一次运行时初始化，单例模式可以有其他的选择，即可以延迟加载。

4. 基于2， 3条，由于单例对象往往存在于DAO层（例如数据访问层，xxxFactory），如果反复的初始化和释放，则会占用很多资源，而使用单例模式将其常驻于内存可以更加节约资源。

5. 静态方法有更高的访问效率。

6. 单例模式很容易被测试。

#### 运用场景
1：不需要维持任何状态，仅仅用于全局访问，此时更适合使用静态类。

2：需要维持一些特定的状态，此时更适合使用单例模式。

### 小结：
实际上，我们在实际项目中一般从懒汉式v3、懒汉式v4、懒汉式v5中，根据实际情况三选一即可，并不是非要选择懒汉式v5作为单例来实现
## Delegate 委派模式
## Strategy 策略模式
## Prototype 原型模式
## Template 模板模式