---
title: 从根儿上理解微服务02
date: 2023-08-07 22:32:00 +0800
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

**[上一篇文章](https://ykfire.github.io/posts/MicroservicesOne/)**中通过引入停车场项目的优化升级，介绍了微服务架构的演进。

今天，咱们继续 👦小K 的故事，再向前推进一步，聊一聊微服务实际开发中无法回避的话题 —— 「服务拆分」。



对于每一个部分，为了方便大家理解和兼顾趣味性，咱们先讲故事，再说结论。

今天的故事从 👦小K 停车场项目发展到 「SOA 架构」 开始讲起，如果有不熟悉的同学，可以先看看**[上一篇文章](https://ykfire.github.io/posts/MicroservicesOne/)**~



## 为什么选择微服务

### 故事

**👦小K的停车场项目团队中存在两个服务，一个是车场服务，另外一个是商城服务，采用的是 「SOA 架构」。**

![image-20230822223545844](/assets/blog_res/2023-08-07-MicroservicesTwo.assets/image-20230822223545844.png)

**随着业务逐渐发展壮大，团队人手逐渐变多，问题也随之暴露。**



**👦小K具体分析思考之后，总结出如下几点问题：**

1. 对于同一个项目而言，项目成员在同一个分支上进行开发，总是互相干扰，经常出现**代码冲突**
2. **服务的小程序端与后台管理系统端存在冗余重复代码，经常出现团队成员重复造轮子**的情况
3. 「车场服务」有时为了给「商城服务」提供相应的**用户接口**，需要不断修改自己的功能，导致二者业务边界逐渐模糊，与最初的规划渐行渐远
4. 主从模式数据库无法满足新的需求，商城相关表与车场相关表耦合在一起，数据库再次出现瓶颈问题
5. 由于服务粒度过粗，当车场或商城服务某一功能发生改动时，整个服务都需要重新部署，**牵一发而动全身**，无法做到**敏捷开发**
6. 出现越来越多**年久失修的 Bug**，原因是担心**耦合度过高**，代码之间互相调用，很难进行修补。

**👦小K心里很清楚，要想解决这么多问题，简单的缝缝补补已经无济于事了，必须得从项目的架构入手！**



**于是，他又开始了他的升级架构之旅～**



### 总结

**为什么需要微服务？简单总结就是：**

1. 技术永远为业务服务，当技术无法满足业务的快速增长时，就需要考虑进行技术升级或项目架构重构，以提高开发效率。而微服务正是为了解决企业长期以来堆积的问题，它是由业务催生的一种技术架构。



**导致传统单体项目向微服务架构进行转型的原因如下：**

1. **敏捷开发问题**
2. 传统单体项目耦合度过于严重，服务粒度太粗，使得开发人员无法专注于业务本身，而需要解决耦合所造成的各种问题，无法做到敏捷开发
3. 项目庞大，提交代码时冲突严重，互相干扰
4. **快速部署问题**
5. 传统部署方式打包时间过长，需要运维人员参与，效率低下
6. 传统单体项目即使是很小的功能修改，也需要重新部署上线，**牵一发而动全身**，严重影响整体开发效率
7. 维护一个庞大的项目，遇到生产环境问题难以定位、难以复现，严重影响经济收益
8. **可扩展性问题：**
9. 模块可扩展性：增加新功能模块时，需要考虑是否影响原有功能模块，模块可扩展性差
10. 资源可扩展性：即使是不同的功能模块，也要共用一套服务器、一套数据库，资源可扩展性差



## 如何划分微服务？

### 故事

**👦小K在想清楚问题之后，便着手开始思考如何升级微服务架构。既然是微服务，那么服务粒度需要再一次降低，也就是得拆分服务。那么应该如何拆分呢？**



👦小K现在面对的可不是 Toy Project，而是他从 0 到 1 搭建的商业化项目，所以这次升级架构一定得**谨小慎微**，划分服务之前需要与开发团队进行多次探讨，确定划分方案。如果随便划分一下服务架构，使得项目重构后反而带来**负收益**呢？



这次，他查阅了很多划分微服务相关的资料，其中提到了很多划分规则。他根据实际业务场景，总结出了如下几点：

1. **划分服务必须满足团队开发需求，以业务为导向，明确业务界限，再进行划分。**
2. 对于 👦小K的项目来说，目前业务界限比较清晰的有这几个模块：「用户服务」、「商品服务」、「车场服务」、「订单服务」、「支付服务」、「优惠券服务」、「推荐服务」、「消息服务」等十余个模块。
3. **最大化地复用微服务，避免重复造轮子。**
4. 👦小K认为，对于「订单服务」、「支付服务」、「优惠券服务」而言，它既可以支撑车场出库需求，又可以支撑车品商城需求，二者代码有很多重复的地方。通过复用服务，减少了重复性的代码。
5. **最小化地变更原有服务。**
6. 👦小K认为，对于车品商城中的「推荐功能」而言，它相对较为独立，处于服务的下游，暂时只被上游「商品服务」调用，只需要将相关的推荐代码抽取出来即可，并且也可为之后新业务提供推荐服务。
7. **确定基础公共服务与业务服务，并从原有项目中抽象公共服务。**
8. 在 👦小K的项目中，除了与业务相关的服务外，还有一些基础服务模块，如「日志监控」、「权限校验」、「搜索服务」等，这些也需要从原有的庞大项目中抽取出来，作为独立的微服务。
9. **尽可能避免循环依赖。若出现循环依赖的情况，需要设计中间层做冗余处理。**
10. **尽可能避免过度拆分微服务，必须结合实际业务场景做划分。**
11. 👦小K想到，如果拆分微服务过细，可能会不适应实际开发需求，反而产生负收益。比如，对于「车场服务」而言，如果进一步拆分「车场地图服务」、「车场详情服务」、「车场预约服务」的话，会导致团队进一步拆分，而车场相关的功能是一个整体，需要成员紧密合作、一同完成需求开发，不适合进行更细粒度的拆分。此外，过度拆分微服务，会导致出现分布式事务问题，复杂度上升，带来不必要的技术成本。



**想清楚这些后，👦小K快速地与团队成员进行了沟通，确定了第一版微服务拆分方案。**



## 总结

**如何拆分微服务？其实对于不同的项目场景，对应有不同的拆分方案。需要项目人员详细地分析项目需求、团队现状、业务边界、业务逻辑等方方面面，拆分的粒度既不能过细，也不能过粗，需要把控好尺度。**



**如下为较为通用的拆分思路：**

1. 从业务模型切入，根据业务边界进行拆分。
2. 确保一个微服务对应一个开发小组或团队。
3. 确保拆分之后所带来的收益大于拆分之前，否则不进行拆分
4. 考虑团队人员配置。在人员较少的情况，服务拆分的粒度不必过细。



## 最后总结

1. 不要为了架构而架构，不要为了 “微服务” 而 “微服务”，不要为了拆分而拆分。**技术永远服务于业务**。
2. 微服务并不是银弹，其也存在缺点；
3. **增加了复杂性**
4. 拆分微服务是一大难题，如何拆分不好，容易产生负收益
5. 其引入了服务治理、负载均衡等基础设施，需要一定维护成本
6. 微服务比单体项目更难理解与上手，并且其一般与 CICD / Devops / 容器化技术配套使用，对技术人员要求高，提升了技术与用人成本
7. **高系统时延**
8. 引入微服务意味着调用链路更长，服务之间需要进行通信，可能带来额外的时延
9. **安全保障问题**
10. 微服务安全性需要考虑的因素比单体架构更多，如服务之间的通信问题



**今天的微服务内容就到这里啦，主要讲解了为什么需要微服务、如何拆分微服务以及微服务的缺点。**