---
title: 使用@Autowired为什么会被IDEA警告？
date: 2023-06-04 23:19:00 +0800
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

 

## 一、导致警告的原因

### 1、初始化问题

Java初始化类的顺序：

> 父类的静态字段 > 父类静态代码块 > 子类静态字段 > 子类静态代码块 > 父类成员变量 > 父类构造代码块 > 父类构造器 > 子类成员变量 > 子类构造代码块 > 子类构造器

而Autowired注入，则要排队到子类构造器以后了，SpringIOC并不会对依赖的bean是否为null做判断，JVM编译时同样也不会有问题，但如果使用不当，运行起来时或许会因为出现空指针异常。



### 2、对IOC容易依赖过强

`@Autowired`由Spring提供，而`@Resource`是JSR-250提供的，它是Java标准。前者会警告，而后者不警告，就是因为前者导致了应用与框架的强绑定，若是换成其他IOC框架，则不能够成功注入了。其实对于这方面，我认为在大多数情况时是不会有什么问题的。



### 3、其他方面

我看到网络上有一些其他方面的总结，比如：依赖过多却不够明显，违反了单一职责原则；不能像构造器那样注入不可变的对象等，这类问题需要结合个人实际开发进行判断。

对于`@Autowired`使用方面，它虽然是**将业务代码和框架进行了强绑定**，但字段注入确实大幅简化了代码。追求完完全全的松耦合其实也过于理想化，**应该在实际使用中追求平衡**，否则将为了过度追求松耦合而得不偿失。



## 二、解决方式

### 1、@Resource注解

除了使用`@Autowired`以外，我们其实也有几种好用的方式。使用`@Resource`替代`@Autiwired`方法是其中一种，只需要改变一个注解，这里就不展示了。



### 2、set方法

```java
@RestController
public class TestController2 {

    ITestService testService;

    /*
     * 基于set注入
     * */
    @Autowired
    public void setTestService(ITestService iTestService) {
        this.testService = iTestService;
    }

    @GetMapping("/status2")
    public Result<?> status() {
        return testService.status();
    }
}
```

这种方法也使用了`@Autowired`注解，但是它是作用于成员变量的Setter函数上，而不是像Fied注入一样作用于成员变量上。



### 3、构造器

```java
@RestController
public class TestController1 {

    ITestService testService;

    /*
    * 基于构造方法的注入
    * */
    public TestController1(ITestService iTestService) {
        this.testService = iTestService;
    }

    @GetMapping("/status1")
    public Result<?> status() {
        return testService.status();
    }
}
```

它的好处在于，采用了构造方法注入，这种方式对对象创建的顺序会有要求，它将避免循环依赖问题。是最可靠的方法。



4、构造器简化版(推荐)

首先，需要引入lombok依赖

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.2</version>
</dependency>
```

随后，我们在创建时就可以使用`@RequiredArgsConstructor`注解，它将帮我们创建构造器，final关键字必不可少。

```java
@RestController
@RequiredArgsConstructor
public class TestController3 {
    /*
     * 用@RequiredArgsConstructor注解，这个使用方式也可以应用于service层
     * */
    private final ITestService testService;


    @GetMapping("/status3")
    public Result<?> status() {
        return testService.status();
    }
}
```

在网上有博主总结了一张表，但因为到处能看到，不知原来出处是哪里。

![image-20230629232411485](/assets/blog_res/2023-06-04-Autowired.assets/image-20230629232411485.png)



## 三、总结

​	在使用中，使用构造方法是比较可行的，加上lombok，其实也可以到达非常简便。

​	但最终还是要根据项目的结构、业务，选择平衡解决的落地方式。