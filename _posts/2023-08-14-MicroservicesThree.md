---
title: 从根儿上理解微服务03
date: 2023-08-14 21:54:00 +0800
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

## 前言 

**[上一篇文章](https://ykfire.github.io/posts/MicroservicesTwo/)**中我详细讲解了为什么需要微服务以及如何划分微服务。今天，咱们正式进入到微服务专栏正题 —— 微服务技术架构。

大家在平常开发的时候，一定或多或少接触过负载均衡、服务发现与注册、服务熔断、服务降级等概念。但是，你有认真地梳理过这些内容吗？如果让你来开发一个微服务，你应该如何开发相关的微服务组件或引入开源解决方案？



今天，我先带大家大概梳理清楚微服务技术架构脉络，使大家对于微服务有一个总体上的认识。



如下为微服务架构思维导图

![image-20230823215558326](/assets/blog_res/2023-08-14-MicroservicesThree.assets/image-20230823215558326.png)

## **服务描述**

在单体项目的开发过程中，很多时候我们采用主流的前后端分离开发模式。

1. 前端请求后端接口
2. 后端将响应 JSON 返回给前端



但是，在微服务项目中，虽然前端仍然只是简单地请求后端，但是后端在处理请求的过程中，可能需要经过不同的微服务。微服务之间是需要进行通信的。



那么在通信之前，我们需要解决一个问题：如何定义当前微服务？更通俗地来说，你的微服务叫什么、服务提供了什么接口、服务接口是如何定义的、服务返回的结果格式是什么、如何解析等。这一系列问题的解决方案就是「服务描述」。



具体的服务描述包括三种方式：

1. RESTful API
2. XML 文件配置
3. IDL 文件配置

这三种方式中，RESTful API 可能是我们日常开发中接触最多的一种形式。即使在单体应用中，我们也经常采用这一方式来规范团队接口定义格式。



IDL 文件也十分常见，它是一种接口描述语言，主要用于 Thrift 与 gRPC 这类跨语言服务框架中，即使是不同语言、不同平台，也可以互相通信。



## **服务发现与注册**

🌟 划重点，重要内容！



当我们实现了多个微服务并定义了相关服务描述之后，应该如何实现互相调用、互相通信呢？这就是服务发现与注册需要解决的问题。



当某一个服务想要调用另外一个服务时，需要知道另一个服务的地址、端口等信息，但是总不可能每一个服务都将所有服务的地址信息记录在本地，这样单服务会存在一定的负载。数据库存储较难维护，也不是一个很好的选择，因为服务的地址信息随时可能发生变化（由于扩缩容、服务下线等原因），这个变化需要被各个服务所感知。



因此，我们需要将这部分信息抽取出来，集中交由某个代理进行统一管理，这就是我们所要提到的「注册中心」。



1. 当服务上线时，需要向注册中心提交自己当前的地址信息，并向注册中心订阅自己所需的服务信息。
2. 当服务下线时，需要通知注册中心，及时删除相关服务信息，并通知其它订阅该服务的服务。
3. 当服务信息发生变化时，注册中心需要及时更改信息。
4. 所有服务需要定时与注册中心进行通信（心跳信息），以确认是否存活。



在日常开发中，较为常见的注册中心有如下几种：

1. Nacos
2. Zookeeper
3. Eureka
4. Consul



## **服务通信**

在单体项目中，不同模块之间可以**通过函数调用的方式**进行交互。



但是，在微服务的世界中，每个微服务最终是以分布式的方式部署于多台服务器上，是一个分布式系统。各个服务实例都是不同的进程。

因此，不同服务之间必须使用 **IPC 进程间通信机制** 进行交互。



如下为两类 IPC 技术：

1. 异步、基于消息的通信
2. 异步通信，客户端不会阻塞等待响应返回
3. 采用消息系统标准协议，如：AMQP / STOMP
4. 开源消息系统：如：RabbitMQ / Kafka / RocketMQ 等
5. 同步的请求 & 响应
6. 同步通信，客户端等待响应时为阻塞状态
7. 两类方式：
8. REST，基于 HTTP 协议
9. Thrift，跨语言、支持多种消息格式



对于服务通信，还需要了解通信的消息格式：文本与二进制

1. 对于文本格式，有 JSON 与 XML 格式
2. 对于二进制，可以采用开源序列化协议，如：Protocol Buffer、Kyro、Hessian等



## **服务路由**

🌟 划重点，重要内容！



### **负载均衡**

1. 在实际的微服务调用中，可能一个微服务对应几十个实例节点，那么对于服务调用者而言，应该如何从中选取节点进行调用呢？这就涉及到了服务节点的负载均衡。
2. 负载均衡，顾名思义，就是为了使得同一服务的不同节点，较为均衡地接收并处理请求。
3. 对于机器性能高、响应速度快、处理请求少的节点，尽可能多地处理请求，发挥最大作用
4. 常见的负载均衡算法有如下几种：
   1. 随机算法
   2. 轮询算法
   3. 加权轮询算法
   4. 最少活跃连接算法
   5. 一致性 Hash 算法



### **分组调用**

通过对 RPC 请求打标签，实现生产环境服务与测试环境服务隔离。



例如：当A、B两个工程师在共同开发一个叫做 UserService 服务时候，A 的 UserService 开发还未完成，处于自测阶段，B 的 UserService 已经开发完成，进入了测试阶段。这时，可以对服务进行分组管理，开发的 UserService 的 group 设置为 dev，B 开发的 UserService 的 group 设置为 test。



### **全链路灰度发布**

当服务全量发布上线之前，我们需要先在一小部分节点上发布服务，进行局部测试。

1. 若测试正常，没有出现问题的话，就继续扩大发布范围，逐渐实现全量更新。
2. 若出现异常，则暂停发布，进行问题排查，这样所造成的负面影响范围较小，不会造成整体的经济损失。



这种部署方式也称为「金丝雀部署」。



## **服务稳定性治理**

🌟 划重点，重要内容！



### **服务限流、降级与熔断**

在我们日常开发中，一定会经常接触到这三个词：限流、降级、熔断。这其实是高并发场景下最常遇到的服务治理问题。



通过这三种方案，确保服务在大多数情况下的「高可用」。

1. 服务限流：在高并发系统中，可能会遇到流量高峰，这时需要通过服务限流，以此限制 QPS，从而起到保护服务、削减流量的作用。
2. 常见限流方式：单实例限流、分布式限流
3. 服务降级：对于微服务项目，为了在高并发场景下保护核心服务，通常会采用 “弃车保帅” 的降级思路，通过丢弃非核心服务请求，削减流量。
4. 服务熔断：当服务出现重大事故、Bug 时，为了防止故障扩散、全链路崩溃、整体服务宕机的情况，需要对服务上游源头进行熔断操作，即直接抛出异常信息，以此在最大程度下保护下游服务。



### **动态超时**

一次微服务请求，可能涉及到几十个服务调用，调用链路十分长。为了防止长时间处理请求，造成请求资源占用的情况，需要引入**超时机制**。



可以有效避免由于长时间的服务调用，造成服务过载，进而引起链路崩溃。



### **请求重试**

当请求失败时，有可能是由于网络阻塞、服务卡顿、TCP 连接异常等原因。这时可以进行适当的请求重试，从而减少请求失败次数，提升服务整体的可用性。



但是，也需要注意设置合理的重试次数，避免重试次数过大、请求累积，进而压垮整个服务。



对于非幂等请求，需要额外小心，防止由于多次重试，造成服务端数据异常问题。



## **服务可观测**

对于微服务系统而言，存在着复杂的链路调用，使得排查线上问题的难度增大。因此，需要建立更加可靠的「可观测系统」，也就是我们日常开发中经常接触的三个概念：**日志系统**、**性能指标**、**链路追踪**。



微服务可观测性并不仅仅是数据的收集、展现、查询，它更加强调数据关联性，帮助我们快速发现、定位问题所在。



## **服务安全保障**

在微服务系统中，我们需要确保服务安全，防止用户敏感信息泄漏。



1. 身份认证：在请求入口（如 API 网关），需要对请求者身份进行校验，较为常规的做法是 Token 校验。
2. 访问控制：对于不同的用户，具有的权限不同，可操控的资源不同，需要进行细粒度、精确到按钮的访问权限控制。具体方案如 RBAC 权限控制。
3. 黑白名单机制：
4. 对于异常流量，需要进行及时的检查与封禁。通过拦截 IP 请求，判断该 IP 是否位于黑名单中，及时拒绝访问
5. 对于开发者、管理员等角色，直接放行，方便开发、测试或管理。





## **服务运维**

微服务架构的升级，往往伴随着容器化技术与 Devops 自动化运维，它们经常搭配使用。



因为当传统的单体应用升级为微服务之后，虽然降低了耦合性，但同时提高了复杂程度，其中就包括运维部署的复杂度。服务的拆分会造成多个服务的打包、测试、部署发布上线，运维的负担大大加重。这时便需要 Devops 解决这一问题。

> Devops 其实是开发 **Development** 与运维 **Operation** 的组合词，这意味着开发与运维的关系变得更加紧密。有了 Devops，开发人员也可解决一键解决运维部署问题，它强调运维部署的自动化流程。



容器化技术则是简化了环境配置操作，解决了环境初始化的问题，使得微服务能够在开发、测试、生产环境之间随意切换。

> Docker 容器化技术能够封装、打包应用程序，将程序以及其依赖文件打包为一个镜像之后，便可在不同的环境快速运行，无需处理配置、依赖问题。



## **总结**

通过本篇文章，我们基本梳理清楚了微服务技术架构的相关知识点。接下来，我们对以上知识点用「一句话」进行总结。



1. **服务描述**：定义服务接口名、接口参数信息、接口响应信息等内容。
2. **服务发现与注册**：抽取服务基本信息到注册中心中，实现服务注册、服务上下线管理、服务调用信息获取功能。
3. **服务通信**：通过异步消息队列或同步请求响应方式，实现 IPC 进程间通信。
4. **服务路由**：负载均衡、分组调用、全链路灰度发布。
5. **服务稳定性治理**：多用于高并发微服务场景，通过服务限流、降级、熔断、请求超时、请求重试等确保服务高可用。
6. **服务可观测**：从日志 Logging、性能指标 Metrics、链路追踪 Tracing 等三个角度，实现服务可观测。
7. **服务安全保障**：在 API 网关入口处，实现身份认证、访问控制、黑白名单过滤机制，保障服务安全，防止数据泄漏。
8. **服务运维**：结合容器化技术与 Devops，实现自动化运维，提升微服务运维效率，开发人员也可承担运维职责，实现敏捷开发。



大家可以顺着知识点进行深入的学习与实践。



