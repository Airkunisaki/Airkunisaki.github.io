---
title: 单例模式
categories: "Java"
tags: "设计模式"
---

# 单例模式的实现

单例模式有最基本的<u>饿汉</u>与<u>懒汉</u>模式，之后为了弥补线程安全及效率问题引进了<u>双重检查加锁</u>模式；

更进阶的，可以使用<u>内部静态类</u>的模式来延迟加载；

最后还有个简单且无法通过反射来获取多个实例的实现方式：<u>枚举类</u>
<!-- more -->

## 饿汉单例

```java
public class Singleton{
  private static Singleton s = new Singleton();
  private Singleton(){}
  public static Singleton getInstance(){
    return s;
  }
}
```
嗯，类加载的时候就已经创建了Singleton静态实例，说明很“饥饿”。
这种方式效率高（线程安全），但是没有延迟加载（一直占用空间）。

## 懒汉式--线程安全

```java
public class Singleton{
  private static Singleton s;
  private Singleton(){}
  public static synchronized Singleton getInstance(){
    if(s == null){
      s = new Singleton();
    }
  return s;
  }
}
```
这种方式安全，达到延迟的目的，但效率太低（为了线程安全，整个方法都加了锁）。

## 双重检查加锁

```java
public class Singleton{
  private static volatile Singleton s;
  private Singleton(){}
  public static Singleton getInstance(){
    if(s == null){
      synchronized（Singleton.class）{
        if(s == null){
          s = new Singleton();
        }
      }
    }
    return s;
  }
}
```
各方面都满足要求（只有第一次创建实例的时候会加锁），java 1.5 后volatile关键字能够禁止代码重排序。
===>ps:为何要加volatile
假设没有关键字volatile的情况下，两个线程A、B，都是第一次调用该单例方法，线程A先执行instance = new Instance()，该构造方法是一个非原子操作，编译后生成多条字节码指令，由于JAVA的指令重排序，可能会先执行instance的赋值操作，该操作实际只是在内存中开辟一片存储对象的区域后直接返回内存的引用，之后instance便不为空了，但是实际的初始化操作却还没有执行，如果就在此时线程B进入，就会看到一个不为空的但是不完整（没有完成初始化）的Instance对象，所以需要加入volatile关键字，禁止指令重排序优化，从而安全的实现单例。

## 内部静态类，Initialization On Demand Holder idiom

```java
public class Singleton{

  private Singleton(){}
  private static class SingletionHolder{
    public static final Singleton s = new Singleton();
  }
  public static Singleton getInstance(){
    return SingletonHolder.s;
  }
}
```
这种方式能保证单例的线程安全的原因：
①静态内部类只有第一次使用才会加载，保证了延迟性；
②因为静态内部类初始化的时候是通过JVM保证线程安全的，所以有线程安全的天然优势。

## 使用枚举（推荐）

```java
enum Singleton {
     INSTANCE;
     public void doSomething(){
          System.out.println("I am writing!");
     }
}

Singleton s = Singleton.INSTANCE;
```
为什么推荐这种呢？因为简单！而且这种方式无法通过反射获取单例类。

P.S.
方法1到4都可以通过反射获取新的单例,如下
```java
Class<Singleton> clz = Singleton.class;
Constructor<Singleton> con == clz.getDeclaredConstructor(new Class[] {});
con.setAccessible(true);
Singleton s3 = con.newInstance(new Object[] {});
```