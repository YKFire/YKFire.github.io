---
title: jar包与war包的区别
date: 2023-06-04 19:54:00 +0800
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

## 一、会产生的奇怪问题

1. 我的一个springboot项目，用mvn install打包成jar，换一台有jdk的机器就直接可以用java -jar 项目名.jar的方式运行，没任何问题，为什么这里不需要tomcat也可以运行了？
2. 然后我打包成war放进tomcat运行，发现端口号变成tomcat默认的8080（我在server.port中设置端口8090）项目名称也必须加上了。

也就是说我在原来的机器的IDEA中运行，**项目接口地址为 ip:8090/listall**,打包放进另一台机器的tomcat就变成了**ip:8080/项目名/listall**。这又是为什么呢？

- 通过jar运行实际上是启动了内置的tomcat,所以用的是应用的配置文件中的端口
- 直接部署到tomcat之后，内置的tomcat就不会启用，所以相关配置**就以安装的tomcat为准**，与应用的配置文件就没有关系了

> 至于为什么会造成这种问题，就要涉及到之前的技术发展进程了

总体来说吧，很多年前，Sun 还在世的那个年代，在度过了早期用 C++写 Html 解析器的蛮荒时期后，有一批最早的脚本程序进入了 cgi 时代，此时的 Sun 决定进军这个领域，为了以示区别并显得自己高大上，于是研发了 servlet 标准，搞出了最早的 jsp。并给自己起了个高大上的称号 JavaEE （ Java 企业级应用标准，其实就是一堆服务器以 http 提供服务...）。

既然是企业级标准那自然得有自己的服务器标准。于是 Servlet 标准诞生，以此标准实现的服务器称为 Servle 容器服务器，Tomcat 就是其中代表，被 Sun 捐献给了 Apache 基金会，那个时候的 Web 服务器还是个高大上的概念，当时的 Java Web 程序的标准就是 War 包(其实就是个 Zip 包)，这就是 War 包的由来。

后来随着服务器领域的屡次进化，人们发现我们为什么要这么笨重的 Web 服务器，还要实现一大堆 Servlet 之外的管理功能，简化一下抽出核心概念 servlet 不是更好吗，最早这么干的似乎是 Jetty，出现了可以内嵌的 Servelet 服务器。

去掉了一大堆非核心功能。后来 tomcat 也跟进了，再后来，本来很笨重的传统 JavaEE 服务器 Jboss 也搞了个 undertow 来凑热闹。正好这个时候微服务的概念兴起，“ use Jar，not War ”。要求淘汰传统 Servlet 服务器的呼声就起来了



## 二、两者的主要区别

1、war是一个web模块，其中需要包括WEB-INF，是可以直接运行的WEB模块；jar一般只是包括一些class文件，在声明了Main_class之后是可以用java命令运行的。

2、war包是做好一个web应用后，通常是网站，打成包部署到容器中；jar包通常是开发时要引用通用类，打成包便于存放管理。

3、war是Sun提出的一种Web应用程序格式，也是许多文件的一个压缩包。这个包中的文件按一定目录结构来组织；classes目录下则包含编译好的Servlet类和Jsp或Servlet所依赖的其它类（如JavaBean）可以打包成jar放到WEB-INF下的lib目录下。

4、JAR文件格式以流行的ZIP文件格式为基础。与ZIP文件不同的是，JAR 文件不仅用于压缩和发布，而且还用于部署和封装库、组件和插件程序，并可被像编译器和 JVM 这样的工具直接使用。

【格式特点】：

- **安全性**　可以对 JAR 文件内容加上数字化签名。这样，能够识别签名的工具就可以有选择地为您授予软件安全特权，这是其他文件做不到的，它还可以检测代码是否被篡改过。
- **减少下载时间**　如果一个 applet 捆绑到一个 JAR 文件中，那么浏览器就可以在一个 HTTP 事务中下载这个 applet 的类文件和相关的资源，而不是对每一个文件打开一个新连接。
- **压缩** JAR 格式允许您压缩文件以提高存储效率。
- **传输平台扩展** Java 扩展框架（Java Extensions Framework）提供了向 Java 核心平台添加功能的方法，这些扩展是用 JAR 文件打包的（Java 3D 和 JavaMail 就是由 Sun 开发的扩展例子）。

> WAR文件就是一个Web应用程序，建立WAR文件，就是把整个Web应用程序（不包括Web应用程序层次结构的根目录）压缩起来，指定一个war扩展名。

【建立的条件】：

- 需要建立正确的Web应用程序的目录层次结构。
- 建立`WEB-INF`子目录，并在该目录下**建立classes与lib两个子目录**。
- 将Servlet类文件放到`WEB-INF\classes`目录下，将Web应用程序所使用Java类库文件（即JAR文件）放到`WEB-INF\lib`目录下。
- 将JSP页面或静态HTML页面放到上下文根路径下或其子目录下。
- 建立`META-INF`目录，并在该目录下建立`context.xml`文件。



## 三、两者的主要特点

Java打包构建后，可以生成两种不同类型的包：JAR包和WAR包。

JAR（Java Archive）包是一种用于打包Java类、资源文件和配置文件的标准Java归档文件格式。它通常用于构建独立的Java应用程序，可以直接通过Java虚拟机（JVM）运行。

WAR（Web Application Archive）包则是一种用于打包JavaWeb应用程序的归档文件格式。它包含了Web应用程序的所有相关资源，例如Servlet、JSP、HTML、CSS、JavaScript等文件，以及配置文件和库文件。WAR包可以被应用服务器（如Tomcat、JBoss等）识别和部署。

主要区别如下：

1. 文件结构：JAR包适用于独立的Java应用程序，而WAR包适用于Web应用程序；
2. 主要内容：JAR包主要包含类文件（.class）、资源文件（.properties、.xml）等，而WAR包除了包含类文件和资源文件外，还包含Web相关文件（Servlet、JSP、HTML、CSS、JavaScript等）；
3. 部署方式：JAR包只需通过Java命令行或者双击运行即可，而WAR包需要将其部署到应用服务器中才能运行；
4. 运行环境：JAR包可以在任何支持Java运行环境（JRE）的操作系统上运行，而WAR包需要在支持JavaWeb应用服务器的操作系统上进行部署和运行。

总的来说，JAR包适用于独立的Java应用程序，而WAR包适用于Web应用程序。选择哪种打包方式取决于你的项目需求和运行环境。



## 四、打包方法

### 1、jar包

- 配置构建工具如maven、gradle，使用其进行构建
- 使用命令行进行打包 jar cf 文件名.jar  文件目录
- 使用开发idea 的 build Artifact 进行打包

### 2、war包

- 配置构建工具如maven、gradle，使用其进行构建 需要`ServletInitializer`类
- 使用开发idea 的 build Artifact 进行打包



## 五、idea相关的功能

### 1、idea build Artifact

在IntelliJ IDEA中，“Build Artifact ”是**用于构建和生成项目输出文件**（例如**JAR包、WAR包**等）的工具。通过Build Artifact，可以将项目编译为可运行的二进制格式，并将其包装为一个或多个文件，便于部署和分发。通常情况下，构建Artifact是项目的最终产品。

在IntelliJ IDEA中配置Build Artifact的步骤如下：

1. 打开项目，然后点击菜单栏的"File"，选择"Project Structure"（或者按下快捷键Ctrl+Alt+Shift+S）。
2. 在Project Structure窗口中，选择"Artifacts"选项卡。
3. 点击"+"按钮，选择你要构建的Artifact类型（例如JAR、WAR等）。
4. 配置Artifact的设置，包括输出路径、依赖关系、源码文件等。
5. 点击"Apply"或"OK"保存配置。

一旦Build Artifact配置完成，你可以使用Build菜单或快捷键，将项目编译为指定的Artifact。IntelliJ IDEA会自动将相关文件打包，并生成Artifact文件。

使用Build Artifact可以让你方便地构建和输出项目的可执行文件，对于部署和发布项目非常有用。



### 2、idea build project

在IntelliJ IDEA中，**"Build Project"是用于编译整个项目的操作**。它会执行以下任务：

1. 检查代码错误：Build Project会对项目的Java源代码进行编译，并检查代码中的语法错误、命名问题、类型错误等。如果代码中存在错误，IDEA会在编译过程中将这些错误显示给你。
2. 生成字节码：Build Project将Java代码编译为字节码（.class文件），这些文件是能够在Java虚拟机（JVM）上执行的二进制文件。
3. 构建依赖：当你修改了项目的源代码、配置文件或资源文件时，Build Project还会重新构建项目的依赖关系。这意味着如果你修改了一个类，那么与该类相关的其他类也会被重新编译。
4. 更新项目输出：Build Project会更新项目的输出文件（例如JAR包、WAR包等），以便能够进行部署和分发。

通常情况下，当你对项目的源代码进行修改后，IntelliJ IDEA会自动执行Build Project操作。你也可以手动触发Build Project，方法是点击菜单栏的"Build"，然后选择"Build Project"（或者按下快捷键Ctrl+F9）。

通过Build Project，**你可以确保项目的代码被正确地编译，并且依赖关系也得到正确的构建，以便进行后续的开发、测试和部署。**