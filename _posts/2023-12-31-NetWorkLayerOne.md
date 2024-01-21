---
title: 计算机网络网络层(一)
date: 2023-12-31 12:32:00 +0800
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



## 计算机网络网络层

前面我们学习了运输层如何为客户端和服务器输送数据的，提供进程端到端的通信。那么下面我们将学习网络层实际上是怎样实现主机到主机的通信服务的。**几乎每个端系统都有网络层这一部分**。所以，网络层必然是很重要的。下面我将花费大量篇幅来介绍一下计算机网络层的知识。



## **网络层概述**



网络层是 OSI 参考模型的第三层，它位于传输层和链路层之间，网络层的主要目的是实现两个端系统之间透明的数据传输。

![image-20240121152628590](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121152628590.png)

网络层的作用从表面看上去非常简单，即将分组从一台主机移动到另外一台主机。为了实现这个功能，网络层需要两种功能



1. **转发**：因为在互联网中有很多路由器，这些路由器是构成互联网的基石，为什么这么说呢？因为路由器最重要的一个功能就是**分组转发**，就是靠着这些路由器，才会把一个个数据包通过链路传输给其他节点。当一个分组到达某路由器的一条输入链路时，该路由器会将分组移动到适当的输出链路。
2. **路由选择**：当分组由发送方流向接收方时，网络层必须选择这些分组的路径。计算这些路径选择的算法被称为**路由选择算法(routing algorithm)**。



也就是说，**转发是指将分组从一个输入链路转移到适当输出链路接口的路由器行为，而路由选择是指确定分组从源到目的地所定位的路径的选择**。我们后面会经常提到转发和路由选择这两个名词。



那么此处就有一个问题，路由器怎么知道有哪些路径可以选择呢？



每台路由器都有一个关键的概念就是**转发表(forwarding table)**。路由器通过检查数据包标头中字段的值，来定位转发表中的项来实现转发。标头中的值即对应着转发表中的值，这个值指出了分组将被转发的路由器输出链路。如下图所示

![image-20240121152826706](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121152826706.png)

上图中有一个 1001 分组到达路由器后，首先会在转发表中进行索引，然后由路由选择算法决定分组要走的路径。每台路由器都有两种功能：**转发和路由选择**。下面我们就来聊一聊路由器的工作原理。



## **路由器工作原理**



下面是一个路由器体系结构图，路由器主要是由 4 个组件构成。

![image-20240121152936804](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121152936804.png)

1. 输入端口：它有很多功能。输入端口查找/转发功能对路由器的交换功能来说至关重要，由路由器的交换结构来决定输出端口，具体来讲应该是查询转发表来确定的。
2. 交换结构：就是将路由器的输入端口连接到它的输出端口。这种交换结构组成了是路由器内部的网络。
3. 输出端口：通过交换结构转发分组，并通过物理层和数据链路层的功能传输分组，因此，输出端口作为输入端口执行反向数据链接和物理层功能。
4. 路由选择处理器：在路由器内执行路由协议，维护路由表并执行网络管理功能。



上面只是这几个组件的简单介绍，其实这几个组件的组成并不像描述的那样简单，下面我们就来深入聊一聊这几个组件。



### 输入端口



上面介绍了输入端口有很多功能，包括**线路终端、数据处理、查找转发**，其实这些功能在输入端口的内部有相应的模块，输入端口的内部实现如下图所示。

![image-20240121153202635](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121153202635.png)

每个输入端口中都有一个路由处理器维护的**路由表的副本**，根据路由处理器进行更新。这个路由表的副本能够使每个输入端口进行切换，而无需经过路由处理器统一处理。**这是一种分散式的切换，这种方式避免了路由选择器统一处理造成转发瓶颈**。



在输入端口处理能力有限的路由器中，输入端口不会进行交换功能，而是由路由处理器统一处理，然后根据路由表查找并将数据包转发到相应的输出端口。



一般这种路由器不是单独的路由器，而是工作站或者服务器充当的路由，这种路由器内部中，路由处理器其实就是 CPU，而输入端口其实只是网卡。



输入端口会根据转发表定位输出端口，然后再会进行分组转发，那么现在就有一个问题，是不是每一个分组都有自己的一条链路呢？如果分组数量非常大，到达亿级的话，也会有亿个输出端口路径吗？



潜意识告诉我显然不是的，来看下面一个例子。



下面是三个输入端口对应了转发表中的三个输出链路的示例

![image-20240121153442588](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121153442588.png)

可以看到，对于这个例子来说，路由器转发表中不需要那么多条链路，只需要四条就够，即对应输出链路 0 1 2 3 。也就是说，能够使用 4 个转发表就可以实现亿级链路。



如何实现呢？



使用这种风格的转发表，路由器分组的地址**前缀(prefix)**会与该表中的表项进行匹配。

![image-20240121153515563](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121153515563.png)

如果存在一个匹配项，那么就会转发到对应的链路上，可能不好理解，我举个例子来说吧。



比如这时有一个分组是 11000011 10010101 00010000 0001100 到达，因为这个分组与 11000011 10010101 00010000 相匹配，所以路由器会转发到 0 链路接口上。如果一个前缀不匹配上面三个输出链路中的一种，那么路由器将向链路接口 3 进行转发。



路由匹配遵循**最长前缀原则(longest prefix matching rule)**，最长匹配原则说的是如果有两个匹配项一个长一个短的话，就匹配最长的。



一旦通过查找功能确定了分组的输出端口后，那么该分组就会进入交换结构。在进入交换结构时，如果交换结构正在被使用，就会阻塞新到的分组，等到交换结构调度新的分组。



### 交换结构



交换结构是路由器的核心，通过交换功能把分组从输入端口转发至输出端口，这就是交换结构的主要功能。交换结构有多种形式，包括**通过内存交换、通过总线交换、通过互联网络进行交换**，下面我们分开来探讨一下。



1. 经过内存交换：最开始的传统计算机就是使用内存交换的，在输入端口和输出端口之间是通过 CPU 进行的。输入端口和输出端口的功能就好像传统操作系统中的 I/O 设备一样。当一个分组到达输入端口时，这个端口会首先以中断的方式向路由选择器发出信号，将分组从输入端口拷贝到内存中。然后路由选择处理器从分组首部中提取目标地址，在转发表中找出适当的输出端口进行转发，同时将分组复制到输出端口的缓存中。



这里需要注意一点，如果内存带宽以每秒读取或者写入 B 个数据包，那么总的交换机吞吐量(数据包从输入端口到输出端口的总速率) 必须小于 B/2。

![image-20240121153749277](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121153749277.png)



1. 经过总线交换：在这种处理方式中，总线经由输入端口直接将分组传送到输出端口，中间不需要路由选择器的干预。总线的工作流程如下：输入端口给分组分配一个标签，然后分组经由总线发送给所有的输出端口，每个输出端口都会判断标签中的端口和自己的是否匹配，如果匹配的话，那么这个输出端口就会把标签拆掉，这个标签只用于交换机内部跨越总线。如果同时有多个分组到达路由器的话，那么只有一个分组能够被处理，其他分组需要再进入交换结构前等待。

![image-20240121153840977](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121153840977.png)



1. 经过互联网络交换：克服单一、共享式总线带宽限制的一种方法是使用一个更复杂的互联网络。如下图所示

![image-20240121153911947](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121153911947.png)



### **输出端口处理**



如下图所示，输出端口取出已经存放在输出端口内存中的分组并将其发送到输出链路上。包括选择和去除排队的分组进行传输，执行所需的链路层和物理层的功能。

![image-20240121154205835](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121154205835.png)

在输入端口中有等待进入交换的排队队列，而在输出端口中有等待转发的排队队列，排队的位置和程度取决于**流量负载、交换结构**的相对频率和线路速率。



随着队列的不断增加，会导致路由器的缓存空间被耗尽，进而使没有内存可以存储溢出的队列，致使分组出现丢包，这就是我们说的在网络中丢包或者被路由器丢弃。



## **何时出现排队**



下面我们通过输入端口的排队队列和输出端口的排队队列来介绍一下可能出现的排队情况。



### **输入队列**



如果交换结构的处理速度没有输入队列到达的速度快，在这种情况下，输入端口将会出现排队情况，到达交换结构前的分组会加入输入端口队列中，以等待通过交换结构传送到输出端口。



为了描述清楚输入队列，我们假设以下情况：



1. 使用网络互联的交换方式；
2. 假定所有链路的速度相同；
3. 在链路中一个分组由输入端口交换到输出端口所花的时间相同，从任意一个输入端口传送到给定的输出端口；
4. 分组按照 FCFS 的方式，只要输出端口不同，就可以进行并行传送。但是如果位于任意两个输入端口中的分组是发往同一个目的地的，那么其中的一个分组将被阻塞，而且必须在输入队列中等待，因为交换结构一次只能传输一个到指定端口。



如下图所示。

![image-20240121154338384](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121154338384.png)

在 A 队列中，输入队列中的两个分组会发送至同一个目的地 X，假设在交换结构正要发送 A 中的分组，在这个时候，C 队列中也有一个分组发送至 X，在这种情况下，C 中发送至 X 的分组将会等待，不仅如此，C 队列中发送至 Y 输出端口的分组也会等待，即使 Y 中没有出现竞争的情况。这种现象叫做**线路前部阻塞(Head-Of-The-Line, HOL)**。



### **输出队列**



我们下面讨论输出队列中出现等待的情况。假设交换速率要比输入/输出的传输速率快很多，而且有 N 个输入分组的目的地是转发至相同的输出端口。在这种情况下，在向输出链路发送分组的过程中，将会有 N 个新分组到达传输端口。因为输出端口在一个单位时间内只能传输一个分组，那么这 N 个分组将会等待。然而在等待 N 个分组被处理的过程中，同时又有 N 个分组到达，所以 ，分组队列能够在输出端口形成。这种情况下最终会因为分组数量变的足够大，从而耗尽输出端口的可用内存。



如果没有足够的内存来缓存分组的话，就必须考虑其他的方式，主要有两种：一种是丢失分组，采用*弃尾(drop-tail)*的方法；一种是删除一个或多个已经排队的分组，从而来为新的分组腾出空间。



**网络层的策略对 TCP 拥塞控制影响很大的就是路由器的分组丢弃策略。**在最简单的情况下，路由器的队列通常都是按照 FCFS 的规则处理到来的分组。由于队列长度总是有限的，因此当队列已经满了的时候，以后再到达的所有分组（如果能够继续排队，这些分组都将排在队列的尾部）将都被丢弃。这就叫做尾部丢弃策略。



通常情况下，在缓冲填满之前将其丢弃是更好的策略。

![image-20240121154624346](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121154624346.png)

如上图所示，A B C 每个输入端口都到达了一个分组，而且这个分组都是发往 X 的，同一时间只能处理一个分组，然后这时，又有两个分组分别由 A B 发往 X，所以此时有 4 个分组在 X 中进行等待。

![image-20240121154730558](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121154730558.png)

等上一个分组被转发完成后，输出端口就会选择在剩下的分组中根据*分组调度(packet scheduleer)*选择一个分组来进行传输，我们下面就会聊到分组传输。



## **分组调度**



现在我们来讨论一下分组调度次序的问题，即排队的分组如何经输出链路传输的问题。我们生活中有无数排队的例子，但是我们生活中一般的排队算法都是*先来先服务(FCFS)*，也是*先进先出(FIFO)*。



### **先进先出**



先进先出就映射为数据结构中的队列，只不过它现在是链路调度规则的排队模型。

![image-20240121154800422](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121154800422.png)

FIFO 调度规则按照分组到达输出链路队列的相同次序来选择分组，先到达队列的分组将先会被转发。在这种抽象模型中，如果队列已满，那么弃尾的分组将是队列末尾的后面一个。



### **优先级排队**



优先级排队是先进先出排队的改良版本，到达输出链路的分组被分类放入输出队列中的优先权类，如下图所示

![image-20240121154823314](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121154823314.png)

通常情况下，每个优先级不同的分组有自己的优先级类，每个优先级类有自己的队列，分组传输会首先从优先级高的队列中进行，在同一类优先级的分组之间的选择通常是以 FIFO 的方式完成。



### **循环加权公平排队**



在**循环加权公平规则(round robin queuing discipline)**下，分组像使用优先级那样被分类。然而，在类之间却不存在严格的服务优先权。循环调度器在这些类之间循环轮流提供服务。如下图所示

![image-20240121154905005](/assets/blog_res/2023-12-31-NetWorkLayer.assets/image-20240121154905005.png)

在循环加权公平排队中，类 1 的分组被传输，接着是类 2 的分组，最后是类 3 的分组，这算是一个循环，然后接下来又重新开始，又从 1 -> 2 -> 3 这个顺序进行轮询。每个队列也是一个先入先出的队列。

这是一种所谓的保持工作排队的规则，就是说如果轮询的过程中发现有空队列，输出端口不会等待分组，而是继续轮询下面的队列。