---
title: Spring Boot自动装配原理
date: 2024-01-14 20:17:00 +0800
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



## Spring Boot自动装配原理

要理解自动装配，主要是要理解透彻两个配置文件的作用，一个是application.yml，一个是spring.factories。下文，让我们具体来讲讲！



# Bean自动装载配置的核心问题



Spring Boot里面的各种**Bean(类对象)能够实现自动装载**，自动的装载帮我们减少了XML的配置，和手动编码进行Bean的加载工作。从而极大程度上帮我们减少了配置量和代码量。

![image-20240201204421266](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201204421266.png)

要实现Bean的自动装载，需要解决两个问题：



1. 如何**保证Bean自动装载的灵活性**？这个问题通过配置文件来解决，在配置A情况下去装载BeanY；在配置B情况下去装载BeanZ。（通常情况下配置A和B会有默认值，来决定默认的装载行为，这样就不需要我们配置了，进一步减少配置量）
2. 如何**保证Bean装载的顺序性**？当BeanA装载完成之后再去装载BeanY，BeanY装载完成之后才去装载BeanX。这个装载顺序问题由@ConditionOnXXXXXXX注解来解决。



# 全局配置文件



SpringBoot使用一个全局的配置文件，配置文件名是固定的；



1. application.properties
2. application.yml



**全局配置文件的作用**：修改SpringBoot自动配置的默认值，**通过配置来影响SpringBoot自动装载行为。**（**很关键**，需要与下文中的spring.factories配置文件进行区分，只有在yml文件中配置了某个依赖，springboot才会去自动装载相应的配置类）



# 配置装载原理源码解析



所有的Spring Boot应用程序都是以SpringApplication.run()作为应用程序入口的。下面我们来一步一步跟踪一下这个函数。

![image-20240201204541851](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201204541851.png)

run方法传入了SpringApplication对象和一些运行期参数。继续向前跟进，我们发现一个类叫做SpringFactoriesLoader，这里面体现了Spring Boot加载配置文件的核心逻辑。

![image-20240201204815450](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201204815450.png)

从上图可以看到：



1. 从META-INF/spring.factories文件夹下加载了spring.factories文件资源
2. 然后读取文件中的ClassName作为值放入Properties



![image-20240201204838487](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201204838487.png)



然后通过反射机制，对spring.factories里面的类资源进行实例化，所以spring.factories文件里面究竟写了什么类？这些类是做什么的？就是我们下一步需要探究的问题了。



**这张图非常形象，把整个装配的流程都画出来了：等看完接下来细节的讲解，再来看这张图可以说是醍醐灌顶。**



![image-20240201204900332](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201204900332.png)



## @EnableAutoConfiguration 作用



SpringBoot入口启动类使用了SpringBootApplication，实际上就是开启了自动配置功能@EnableAutoConfiguration。

![image-20240201204933992](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201204933992.png)

可以把 @SpringBootApplication看作是 @Configuration、@EnableAutoConfiguration、@ComponentScan 注解的集合。根据 SpringBoot 官网，这三个注解的作用分别是：



1. @EnableAutoConfiguration：启用 SpringBoot 的自动配置机制
2. @Configuration：允许在上下文中注册额外的 bean 或导入其他配置类
3. @ComponentScan：扫描被@Component (@Service,@Controller)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。如下图所示，容器中将排除TypeExcludeFilter和AutoConfigurationExcludeFilter。



@EnableAutoConfiguration 是实现自动装配的重要注解，我们以这个注解入手。

```java
@Target({ElementType.TYPE})

@Retention(RetentionPolicy.RUNTIME)

@Documented

@Inherited

@AutoConfigurationPackage //作用：将main包下的所有组件注册到容器中

@Import({AutoConfigurationImportSelector.class}) //加载自动装配类 xxxAutoconfiguration

public @interface EnableAutoConfiguration {

String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

Class<?>[] exclude() default {};

String[] excludeName() default {};

}
```

我们现在重点分析下AutoConfigurationImportSelector 类到底做了什么？



### AutoConfigurationImportSelector:加载自动装配类



AutoConfigurationImportSelector类的继承体系如下：

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

}

public interface DeferredImportSelector extends ImportSelector {

}

public interface ImportSelector {
String[] selectImports(AnnotationMetadata var1);
}
```



可以看出，AutoConfigurationImportSelector 类实现了 ImportSelector接口，也就实现了这个接口中的 selectImports方法，该方法主要用于获取所有符合条件的类的全限定类名，这些类需要被加载到 IoC 容器中。



这里我们需要重点关注一下getAutoConfigurationEntry()方法，这个方法主要负责加载自动配置类的。



调用图：

![image-20240201205237299](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201205237299.png)

现在我们结合getAutoConfigurationEntry()的源码来详细分析一下：

![image-20240201205254182](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201205254182.png)

1. 判断自动装配开关是否打开。默认spring.boot.enableautoconfiguration=true，可在 application.properties 或 application.yml 中设置

2. 获取EnableAutoConfiguration注解中的 exclude 和 excludeName

![image-20240201205327686](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201205327686.png)

3. 获取需要自动装配的所有配置类，读取META-INF/spring.factories



spring-boot/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories

![image-20240201205355278](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201205355278.png)

从下图可以看到这个文件的配置内容都被我们读取到了。XXXAutoConfiguration的作用就是按需加载组件。

![image-20240201205416150](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201205416150.png)

所有 Spring Boot Starter 下的META-INF/spring.factories都会被读取到。



如果，我们自己要创建一个 Spring Boot Starter，这一步是必不可少的。

![image-20240201205438799](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201205438799.png)

spring.factories中这么多配置，每次启动都要全部加载么?



我们 debug 到后面你会发现，configurations 的值变小了:![image-20240201205457678](/assets/blog_res/2024-01-21-AutomaticAssembly.assets/image-20240201205457678.png)

因为，这一步有经历了一遍筛选，@ConditionalOnXXX 中的所有条件都满足，该类才会生效

```java
@Configuration
// 检查相关的类：RabbitTemplate 和 Channel是否存在
// 存在才会加载
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
}
```



//具体的装配检查注解，可以看下方的文档。

简单来说：SpringFactoriesLoader会以@EnableAutoConfiguration的包名和类名org.springframework.boot.autoconfigure.EnableAutoConfiguration为Key查找spring.factories文件,并将value中的类名实例化加载到Spring Boot应用中。

spring.factories文件中的每一行都是一个自动装配类。



# Spring boot的自动装配实现原理简述



如果上述原理细节无法理解，面试只需要掌握以下这种程度！



自动装配可以简单理解为：**通过注解或者一些简单的配置就能在 Spring Boot 的帮助下实现某块功能。**



Springboot 所有自动配置都是在启动时候被扫描并加载：所有的自动配置类都在 spring.factories 里面，但是不一定生效，需要判断条件是否成立，**只要导入了对应的starter，就会有对应的启动器，有了启动器，我们的自动装配就会生效，然后就配置成功！**



## 补充：Starter是什么？



Spring Boot Starter的中文名为"**启动器**"。

"Starter"是Spring Boot中的一个概念，它是一种**特殊的依赖包**，旨在简化和快速配置特定功能的应用程序。Starter包含了一组预配置的依赖项，使得我们可以很方便地引入和使用某个特定功能或技术栈。



Spring Boot的Starter遵循一种命名约定，通常以spring-boot-starter-*的形式命名，例如：



1. spring-boot-starter-web：用于构建Web应用程序的启动器，包含了常用的Web开发所需的依赖项，如Spring MVC、Tomcat等。
2. spring-boot-starter-data-jpa：用于支持使用JPA进行数据库访问的启动器，包含了JPA、Hibernate、数据库驱动等依赖项。
3. spring-boot-starter-test：用于编写测试的启动器，包含了JUnit、Mockito等测试框架的依赖项。
