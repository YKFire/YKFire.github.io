---
title: 详解Java类中serialVersionUID
date: 2022-10-22 22:15:00 +0800
categories: [技术科普]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---

# 详解Java类中serialVersionUID

> 一直以来在制作项目的过程中，使用Mybatis-X插件生成的Java实体类都自动会实现 **Serializable**这个接口，并自动生成了*serialVersionUID*这一属性
>
>  最近得闲查阅了相关资料，了解了其存在的作用，便决定编写相应笔记记录下来

## 一、serialVersionUID简介

​	*serialVersionUID*属性是用来序列的**标识符**/反序列化的对象序列化类

​	序列化运行时**可与每个可序列化的类关联一个版本号**，称为*serialVersionUID*，在反序列化期间使用该版本号来验证序列化对象的发送者和接收者是否已加载了该对象的与序列化兼容的类。

## 二、serialVersionUID的作用

​	*serialVersionUID*适用于**Java的序列化机制**。简单来说，Java的序列化机制是通过判断类的*serialVersionUID*来验证版本一致性的。在进行反序列化时，**JVM**会把传来的字节流中的*serialVersionUID*与本地相应实体类的*serialVersionUID*进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常，即是`InvalidClassExceptions`。

## 三、serialVersionUID的实现方式

### 1、手动定义

> 当我们在编写程序的时候，可以为可序列化的类显式地声明一个*serialVersionUID*，并且其必须使用**static、final**两个关键字进行声明，数据类型必须为**long**

```java
private static final long serialVersionUID = 1L;
```

### 2、自动生成

> 当一个类实现Serializable类接口，如果没有显示定义*serialVersionUID*，程序序列化运行时会根据**包名，类名，继承关系，非私有的方法和属性，以及参数，返回值等诸多因子计算得出的，极度复杂生成的一个64位的哈希字段**作为默认的*serialVersionUID*值。

### 3、建议

>强烈建议所有可序列化的类**显式声明**`serialVersionUID`值，因为默认*serialVersionUID*计算**对类详细信息高度敏感**，而类详细信息可能会根据编译器的实现而有所不同，因此可能在`InvalidClassExceptions`反序列化期间导致意外情况
>
>因此，为了保证*serialVersionUID*在**不同Java编译器**实现之间的值一致，可序列化的类**建议声明一个显式*serialVersionUID*值**。还强烈建议在*serialVersionUID*的声明中尽可能使用**private修饰符**，因为此类声明仅适用于立即声明的类-*serialVersionUID*字段**作为继承成员没有用**。

## 四、应用场景

### 1、代码实现

序列化实体类：

```java
public class Persion implements Serializable {
 
    private static final long serialVersionUID = 4359709211352400087L;
    public Long id;
    public String name;
    public final String userName;
 
    public Persion(Long id, String name){
        this.id = id;
        this.name = name;
        userName = "dddbbb";
    }
 
    public String toString() {
        return id.toString() + "--" + name.toString();
    }
}
```

序列化过程：

```java
public class SerialTest {
 
    public static void main(String[] args) {
        Persion p = new Persion(1L, "陈俊生");
        System.out.println("person Seria:" + p);
        try {
            FileOutputStream fos = new FileOutputStream("Persion.txt");
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            oos.writeObject(p);
            oos.flush();
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

反序列化过程：

```java
public class DeserialTest {
 
    public static void main(String[] args) {
        Persion p;
        try {
            FileInputStream fis = new FileInputStream("Persion.txt");
            ObjectInputStream ois = new ObjectInputStream(fis);
            p = (Persion) ois.readObject();
            ois.close();
            System.out.println(p.toString());
            System.out.println(p.userName);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### 2、具体情况

​	Persion类**序列化之后**，假定从**A端传递到B端**，然后在**B端进行反序列化**,之后所有的情况都建立在这基础之上

#### 情况一

>在序列化Persion和反序列化Persion的时候A和B端都需要一个相同的类，如果两处的*serialVersionUID*不一致，会产生什么样的效果呢？

​	最终报出`InvalidClassExceptions`异常

```tex
java.io.InvalidClassException:serializable.Persion; local class incompatible: stream classdesc serialVersionUID = 4359709211352400087, local class serialVersionUID = 4359709211352400082
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:616)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1843)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1713)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2000)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1535)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:422)
```

#### 情况二

> 假设两处*serialVersionUID*一致，如果A端增加一个字段，然后序列化，而B端不变，然后反序列化，会是什么情况呢?

​	B端进行反序列化且反序列化正常，A端中**新增加的字段会丢失**（被B端忽略）

#### 情况三

> 假设两处*serialVersionUID*一致，如果B端丢失一个字段，A端不变，会是什么情况呢?

​	B端进行反序列化且反序列化正常，A端会**减少与B段丢失的相同字段**（被B端忽略）

#### 情况四

> 假设两处*serialVersionUID*一致，如果B端增加一个字段，A端不变，会是什么情况呢?

​	B端进行反序列化且反序列化正常，A端**获得与B段增加的相同字段**

## 五、总结

​	实现*serialVersionUID*接口的目的是**为实现类可持久化**，比如在网络传输或本地存储，**为系统的分布和异构部署提供先决条件**。若没有序列化，现在我们所熟悉的远程调用，对象数据库都不可能存在。