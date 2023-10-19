---
title: 计算机网络应用层协议(二)
date: 2023-10-01 23:11:00 +0800
categories: [计算机网络]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---



## **应用层协议**

### **WWW 和 HTTP**



万维网 (WWW, World Wide Web)是将互联网中的信息以超文本的形式展现的系统，用来显示 WWW 结果的客户端被称为 Web 浏览器。通过浏览器，我们无需关注想要访问的内容在哪个服务器上，我们只需要知道我们想访问的内容就可以了。

![image-20231019231241286](/C:/Users/legion/AppData/Roaming/Typora/typora-user-images/image-20231019231241286.png)

万维网定义了三个比较重要的概念：



1. URI：定义了访问信息的手段和位置
2. HTML：定义了信息的表现形式
3. HTTP：定义了 WWW 的访问规范



#### **URI / URL**



URI (Uniform Resource Identifier)中文名称是统一资源标识符，使用它就能够唯一地标识互联网上的资源。



URL (Uniform Resource Locator)中文名称是统一资源定位符，也就是我们俗称的网址，它实际上是 URI 的一个子集。



URI 不仅包括 URL，还包括 URN（统一资源名称），它们之间的关系如下

![image-20231019231323060](/C:/Users/legion/AppData/Roaming/Typora/typora-user-images/image-20231019231323060.png)



而且 URI 已经不局限于标识互联网资源，它可以作为所有资源的识别码。



#### **HTML**



HTML 称为超文本标记语言，是一种标识性的语言。它包括一系列标签．通过这些标签可以将网络上的文档格式统一，使分散的 Internet 资源连接为一个逻辑整体。HTML 文本是由 HTML 命令组成的描述性文本，HTML 命令可以说明文字，图形、动画、声音、表格、链接等。



#### **HTTP**



Web 的应用层协议就是HTTP (HyperText Transfer Protocol, HTTP)， 超文本传输协议，它是 Web 的核心协议。



#### **Web 页面**



Web 页面也叫做 Web Page，它是由无数个对象组成，一个对象就是一个文件，这个文件可以是 HTML 文件、一个图片、一段 Java 应用程序等，它们都可以通过 URI 来唯一定位。一个 Web 页面包含了很多对象，Web 页面可以说是对象的集合体。



#### **浏览器**



就如同各大邮箱使用电子邮件传送协议 SMTP 一样，浏览器是使用 HTTP 协议的主要载体，说到浏览器，你能想起来哪几个？随着网景大战结束后，浏览器迅速发展，至今已经出现过的浏览器主要有 IE、Firefox、Chrome、Safari、Opera、Netscape、傲游等。



![image-20231019231436045](/C:/Users/legion/AppData/Roaming/Typora/typora-user-images/image-20231019231436045.png)



#### **Web 服务器**



Web 服务器的正式名称叫做 Web Server，Web 服务器可以向浏览器等 Web 客户端提供文档，也可以放置网站文件，让全世界浏览；可以放置数据文件，让全世界下载。目前最主流的三个 Web 服务器是 Apache、 Nginx 、IIS。



#### **CDN**



CDN 是一种内容分发网络，它应用了 HTTP 协议里的缓存和代理技术，代替源站响应客户端的请求。CDN 是构建在现有网络基础之上的网络，它依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN 的关键技术主要有内容存储和分发技术。



打比方说你要去亚马逊上买书，之前你只能通过购物网站购买后从美国发货过海关等重重关卡送到你的家里，现在在中国建立一个亚马逊分基地，你就不用通过美国进行邮寄，从中国就能把书尽快给你送到。



#### **WAF**



WAF 是一种 **Web应用程序防护系统 (Web Application Firewall，简称 WAF)**，它是一种通过执行一系列针对 HTTP / HTTPS 安全策略来专门为 Web 应用提供保护的一款产品，它是应用层面的防火墙，专门检测 HTTP 流量，是防护 Web 应用的安全技术。



WAF 通常位于 Web 服务器之前，可以阻止如 SQL 注入、跨站脚本等攻击，目前应用较多的一个开源项目是 ModSecurity，它能够完全集成进 Apache 或 Nginx。



#### **WebService**



WebService 是一种 Web 应用程序，WebService 是一种跨编程语言和跨操作系统平台的远程调用技术。



WebService 是一种由 W3C 定义的应用服务开发规范，使用 client-server 主从架构，通常使用 WSDL 定义服务接口，使用 HTTP 协议传输 XML 或 SOAP 消息，它是一个基于 Web（HTTP）的服务架构技术，既可以运行在内网，也可以在适当保护后运行在外网。



下面我们就来认识一下应用层常见的几个协议，一些我们常用且重要的协议比如 HTTP、DNS 会在后面详细介绍，目前可以作为了解。



### **HTTP**



**HTTP 是一个在网络中专门在两点之间传输文字、图片、音频、视频等超文本数据的约定和规范**。HTTP 是一种应用层协议，它使用 TCP 作为运输层协议。



#### **HTTP 请求响应过程**



让我们通过一个例子来探讨一下 HTTP 的请求响应过程，我们假设访问的 URL 地址为



[http://www.someSchool.edu/someDepartment/home.index](http://www.someschool.edu/someDepartment/home.index)，当我们输入网址并点击回车时，浏览器内部会进行如下操作

1. DNS 服务器会首先进行域名的映射，找到访问 [www.someSchool.edu](http://www.someschool.edu/) 所在的 IP 地址，然后 HTTP 客户端进程在 80 端口发起一个到服务器 [www.someSchool.edu](http://www.someschool.edu/) 的 TCP 连接（80 端口是 HTTP 的默认端口）。在客户和服务器进程中都会有一个套接字与其相连。



1. HTTP 客户端通过它的套接字向服务器发送一个 HTTP 请求报文。该报文中包含了路径 someDepartment/[home.index](http://home.index/) 的资源（我们后面会详细讨论 HTTP 请求报文）。



1. HTTP 服务器通过它的套接字接受该报文，进行请求的解析工作，并从其存储器 ( RAM 或磁盘 ) 中检索出对象 [www.someSchool.edu/someDepartment/home.index](http://www.someschool.edu/someDepartment/home.index)，然后把检索出来的对象进行封装，封装到 HTTP 响应报文中，并通过套接字向客户进行发送。



1. HTTP 服务器随即通知客户端断开 TCP 连接，实际上是需要等到客户接受完响应报文后才会断开 TCP 连接。



1. HTTP 客户端接受完响应报文后，TCP 连接关闭。HTTP 客户端从响应体里提取 HTML 响应文件，并检查该 HTML 文件，然后循环检查报文中其他内部对象。



1. 检查完成后，HTTP 客户端会通过浏览器渲染把对应的资源通过显示器呈现给用户。



至此，键入网址再按下回车的过程就结束了。上述过程描述的是一种简单的请求-响应全过程，真实的请求-响应情况要比上面描述的过程复杂很多。



#### **HTTP 请求特征**



从上面整个过程中我们可以总结出 HTTP 进行分组传输是具有以下特征：



1. 支持客户 - 服务器模式的问答形式。



1. 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有 GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于 HTTP 协议简单，使得 HTTP 服务器的程序规模小，因而通信速度很快。



1. 灵活：HTTP 允许传输任意类型的数据对象。正在传输的类型由 Content-Type 加以标记。



1. 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。



1. 无状态：HTTP 协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。



#### **持久连接和非持久连接**



我们上面描述的 HTTP 请求响应过程就是一种非持久性连接，因为每次 TCP 在传递完报文后，都会关闭 TCP 连接，每个 TCP 连接只传输一个请求报文和响应报文。



非持久性连接有一些缺点：



1. 首先必须为每个请求的对象建立和维护一个全新的连接。



1. 对于每个这样的连接来说，在客户端和服务器中都要分配 TCP 的缓冲区，这无疑给 Web 服务器带来了严重的负担。因为一台 Web 服务器可能要同时服务于数百甚至上千个客户请求。



在采用 HTTP/1.1 持久连接的情况下，服务器在发送响应后会保持该 TCP 连接，后续的请求和响应报文能够通过相同的连接进行传输。如果在一段时间内（可配置）该连接并未再次使用，HTTP 服务器就会断开连接。



#### **HTTP 报文格式**



我们上面描述了一下 HTTP 的请求响应过程，相信你对 HTTP 有了更深的认识，下面我们就来一起认识一下 HTTP 的报文格式是怎样的。



HTTP 协议主要由三大部分组成：



1. 起始行（start line）：描述请求或响应的基本信息。



1. 头部字段（header）：使用 key-value 形式更详细地说明报文。



1. 消息正文（entity）：实际传输的数据，它不一定是纯文本，可以是图片、视频等二进制数据。



其中起始行和头部字段并成为请求头或者响应头，统称为 Header；消息正文也叫做实体，称为 body。HTTP 协议规定每次发送的报文必须要有 Header，但是可以没有 body，也就是说头信息是必须的，实体信息可以没有。而且在 header 和 body 之间必须要有一个空行（CRLF）。如果用一幅图来表示一下 HTTP 请求的话，我觉得应该是下面这样

![image-20231019231806120](/C:/Users/legion/AppData/Roaming/Typora/typora-user-images/image-20231019231806120.png)

如果细化一点的话，那就是下面这样：

![image-20231019231820756](/C:/Users/legion/AppData/Roaming/Typora/typora-user-images/image-20231019231820756.png)

这幅图需要注意一下，如果使用 GET 方法，是没有实体体的，如果你使用的是 POST 方法，才会有实体体。当用户提交表单时，HTTP 客户端通常使用 POST 方法；与此相反，HTML 表单的获取通常使用 GET 方法。HEAD 方法类似于 GET 方法，只不过 HEAD 方法不会返回对象。



下面我们来看一下 HTTP 响应报文

![image-20231019231846336](/C:/Users/legion/AppData/Roaming/Typora/typora-user-images/image-20231019231846336.png)

可以看到，请求报文和响应报文只有请求头是不同的，其他信息均一致。



请求报文请求行：

GET /some/page.html HTTP/1.1

响应报文：

HTTP/1.1 200 OK



### **Cookie 和 Session**



HTTP 协议是一种无状态协议，即每次服务端接收到客户端的请求时，都是一个全新的请求，服务器并不知道客户端的历史请求记录；但是一些网站却需要记住客户信息，比如电商网站客户登录之后选购商品，客户把商品加入购物车之后会跳转到结算页面，这个时候网站需要知道客户信息，如果使用无状态的 HTTP 协议，那么客户根本无法结算。



Session 和 Cookie 的主要目的就是为了弥补 HTTP 的无状态特性。



#### **Session 是什么**



客户端请求服务端，服务端会为这次请求开辟一块内存空间，这个对象便是 Session 对象，存储结构是一个 Map 映射，具体一点是 ConcurrentHashMap。Session 弥补了 HTTP 无状态特性，服务器可以利用 Session 存储客户端在同一个会话期间的一些操作记录。



#### **Session 如何判断是否是同一会话**



服务器第一次接收到请求时，开辟了一块 Session 空间（创建了 Session 对象），同时生成一个 sessionId ，并通过响应头的 Set-Cookie：JSESSIONID=XXXXXXX 命令，向客户端发送要求设置 Cookie 的响应； 客户端收到响应后，在本机客户端设置了一个 JSESSIONID=XXXXXXX 的 Cookie 信息，该 Cookie 的过期时间为浏览器会话结束；

![image-20231019232009055](/C:/Users/legion/AppData/Roaming/Typora/typora-user-images/image-20231019232009055.png)

接下来客户端每次向同一个网站发送请求时，请求头都会带上该 Cookie信息（包含 sessionId ），然后服务器通过读取请求头中的 Cookie 信息，获取名称为 JSESSIONID 的值，得到此次请求的 sessionId。



Session 机制有个缺点，比如 A 服务器存储了 Session，就是做了负载均衡后，假如一段时间内 A 的访问量激增，会转发到 B 进行访问，但是 B 服务器并没有存储 A 的 Session，会导致 Session 的失效。



#### **Cookies 是什么**



HTTP 协议中的 Cookie 包括 Web Cookie 和浏览器 Cookie，它是服务器发送到 Web 浏览器的一小块数据。服务器发送到浏览器的 Cookie，浏览器会进行存储，并与下一个请求一起发送到服务器。通常，它用于判断两个请求是否来自于同一个浏览器，例如用户保持登录状态。



HTTP Cookie 机制是 HTTP 协议无状态的一种补充和改良。



Cookie 主要用于下面三个目的



1. 会话管理：登陆、购物车、游戏得分或者服务器应该记住的其他内容。



1. 个性化：用户偏好、主题或者其他设置。



1. 追踪：记录和分析用户行为。



Cookie 曾经用于一般的客户端存储。虽然这是合法的，因为它们是在客户端上存储数据的唯一方法，但如今建议使用现代存储 API。Cookie 随每个请求一起发送，因此它们可能会降低性能（尤其是对于移动数据连接而言）。



#### **创建 Cookie**



当接收到客户端发出的 HTTP 请求时，服务器可以发送带有响应的 Set-Cookie 标头，Cookie 通常由浏览器存储，然后将 Cookie 与 HTTP 标头一同向服务器发出请求。



#### **Set-Cookie 和 Cookie 标头**



Set-Cookie HTTP 响应标头将 cookie 从服务器发送到用户代理。下面是一个发送 Cookie 的例子：

![image-20231019232043372](/C:/Users/legion/AppData/Roaming/Typora/typora-user-images/image-20231019232043372.png)

此标头告诉客户端存储 Cookie。



现在，随着对服务器的每个新请求，浏览器将使用 Cookie 头将所有以前存储的 Cookie 发送回服务器。



有两种类型的 Cookies，一种是 Session Cookies，一种是 Persistent Cookies，如果 Cookie 不包含到期日期，则将其视为会话 Cookie。会话 Cookie 存储在内存中，永远不会写入磁盘，当浏览器关闭时，此后 Cookie 将永久丢失。如果 Cookie 包含有效期 ，则将其视为持久性 Cookie。在到期指定的日期，Cookie 将从磁盘中删除。



还有一种是 Cookie 的 Secure 和 HttpOnly 标记，下面依次来介绍一下：



#### **会话 Cookies**



上面的示例创建的是会话 Cookie ，会话 Cookie 有个特征，客户端关闭时 Cookie 会删除，因为它没有指定Expires 或 Max-Age 指令。



但是，Web 浏览器可能会使用会话还原，这会使大多数会话 Cookie 保持永久状态，就像从未关闭过浏览器一样。



#### **永久性 Cookies**



永久性 Cookie 不会在客户端关闭时过期，而是在特定日期 ( Expires ) 或特定时间长度 ( Max-Age ) 外过期。例如

Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;



#### **对 Cookie 的争论**



尽管 Cookie 能够简化用户的网络活动，但是 Cookie 的使用存在争议，因为不少人认为它对用户是一种侵权行为。因为结合 Cookie 和用户提供的账户信息，Web 站点可以知道更多关于用户的信息。
