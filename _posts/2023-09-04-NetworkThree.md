---
title: 计算机网络发展史(三)
date: 2023-09-04 21:50:00 +0800
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



## **网络核心概念**

计算机网络中，有一些核心概念是大家需要知道和理解的。

### **传输方式**



网络根据传输方式可以进行分类，一般分为**面向连接型和面向无连接型**。



1. 面向连接型中，在发送数据之前，需要在主机之间建立一条通信线路。
2. 面向无连接型则不要求建立和断开连接，发送方可用于任何时候发送数据。接收端也不知道自己何时从哪里接收到数据。



### **分组交换**



先来介绍一下端系统的概念。



处在互联网边缘部分的机器，也就是互联网终端主机，它们官方一点的话术就是端系统。



在互联网应用中，每个终端系统都可以彼此交换信息，这种信息也被称为报文(Message)，报文是一个超集的概念，它可以包括你想要的任何东西，比如文字、数据、电子邮件、音频、视频等。为了从源目的地向端系统发送报文，需要把长报文切分为一个个小的数据块，这种数据块称为分组(Packets)，也就是说，报文是由一个个小块的分组组成。

![image-20231009223111543](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009223111543.png)

在分组的传输过程中，每个分组都要经过通信链路和分组交换机，分组要在端系统之间传输需要经过一定的时间，如果两个端系统之间需要传输的分组为 L 比特，链路的传输速率问 R 比特/秒，那么传输时间就是 L / R秒。



一个端系统需要经过交换机给其他端系统发送分组，当分组到达分组交换机时，交换机会如何操作？交换机会直接进行转发吗？不是的，交换机可没有这么无私，你想让我帮你转发分组？好，首先你需要先把整个分组数据都给我，我再考虑给你发送的问题，这就是**存储转发传输**。



#### **存储转发传输**



下面是一个存储转发传输的示意图。

![image-20231009223207830](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009223207830.png)

由上图可以看出，分组 1、2、3 向交换器进行分组传输，并且交换机已经收到了分组 1 发送的比特，此时交换机会直接进行转发吗？答案是不会的，交换机会把你的分组先缓存在本地。这就和考试作弊一样，一个学霸要经过学渣 A 给学渣 B 传答案，学渣 A 在收到答案后，它可能直接把卷子传过去吗？学渣 A 说，等我先把答案抄完（保存功能）后再把卷子给你，当然一个及其有素质的学渣就另说了。



#### **排队时延和分组丢失**



什么？你认为交换机只能和一条通信链路进行相连？那你就大错特错了，这可是交换机啊，怎么可能只有一条通信链路呢？



所以我相信你一定能想到这个问题，当多个端系统同时给交换器发送分组，一定存在顺序到达和排队问题。事实上，对于每条相连的链路，该分组交换机都会有一个输出缓存(output buffer)和 输出队列(output queue) 与之对应，它用于存储路由器准备发往每条链路的分组。如果到达的分组发现路由器正在接收其他分组，那么新到达的分组就会在输出队列中进行排队，这种等待分组转发所耗费的时间也被称为**排队时延**，上面提到分组交换器在转发分组时会进行等待，这种等待被称为**存储转发时延**，所以我们现在了解到的有两种时延，但是其实是有四种时延。这些时延不是一成不变的，其变化程序取决于网络的拥塞程度。



因为队列是有容量限制的，当多条链路同时发送分组导致输出缓存无法接受超额的分组后，这些分组会丢失，这种情况被称为丢包(packet loss)，到达的分组或者已排队的分组将会被丢弃。



下图说明了一个简单的分组交换网络。

![image-20231009223301619](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009223301619.png)

下面来一个情景模拟：假定主机 A 和 主机 B 要向主机 E 发送分组，主机 A 和 B 首先通过 100 Mbps 以太网链路将其数据包发送到第一台路由器，然后路由器将这些数据包定向到 15 Mbps 的链路。如果在较短的时间间隔内，数据包到达路由器的速率（转换为每秒比特数）超过 15 Mbps，则在数据包在链路输出缓冲区中排队之前，路由器上会发生拥塞，然后再传输到链路上。例如，如果主机 A 和主机 B 背靠背同时发了 5 包数据，那么这些数据包中的大多数将花费一些时间在队列中等待。实际上，这种情况与许多普通情况完全相似，例如，当我们排队等候银行出纳员或在收费站前等候时。



#### **转发表和路由器选择协议**



我们刚刚讲过，路由器和多个通信线路进行相连，如果每条通信链路同时发送分组的话，可能会造成排队和丢包的情况，然后分组在队列中等待发送，现在我就有一个问题问你，队列中的分组发向哪里？这是由什么机制决定的？

换个角度想问题，路由的作用是什么？其实是**把不同端系统中的数据包进行存储和转发** 。在互联网中，每个端系统都会有一个 IP 地址，当主机发送分组时，会在分组的首部加上主机的 IP 地址。每台路由器都会有一个转发表(forwarding table)，当一个分组到达路由器后，路由器会检查分组中目的地址的一部分，并用目的地址搜索转发表，以找出适当的传输链路，然后映射到输出链路实现转发操作。



那么问题来了，路由器内部是怎样设置转发表的呢？详细的我们后面会讲到，这里只是说个大概，路由器内部也是具有路由选择协议的，用于自动设置转发表。



### **电路交换**



在计算机网络中，另一种通过网络链路和路由进行数据传输的另外一种方式就是电路交换(circuit switching)。电路交换在资源预留上与分组交换不同，这是什么意思呢？



就是**分组交换不会预留每次端系统之间交互分组的缓存和链路传输速率**，所以每次都会进行排队传输；**而电路交换会预留这些信息**。一个简单的例子帮助你理解：这就好比有两家餐馆，餐馆 A 需要预定而餐馆 B 不需要预定，对于可以预定的餐馆 A，我们必须先提前与其进行联系，但是当我们到达目的地时，我们能够立刻入座并选菜。而对于不需要预定的那家餐馆来说，你可能不需要提前联系，但是你必须承受到达目的地后需要排队的风险。



下面显示了一个电路交换网络

![image-20231009223806994](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009223806994.png)

在这个网络中，4 条链路用于 4 台电路交换机。这些链路中的每一条都有 4 条电路，因此每条链路能支持 4 条并行的链接。每台主机都与一台交换机直接相连，当两台主机需要通信时，该网络在两台主机之间创建一条专用的端到端的链接(end-to-end connection)。



### **分组交换和电路交换的对比**



电路交换的支持者经常说分组交换不适合实时服务，因为它的端到端时延时不可预测的。而分组交换的支持者却认为分组交换提供了比电路交换更好的带宽共享；它比电路交换更加简单、更有效，实现成本更低。但是现在的趋势更多的是朝着分组交换的方向发展。



### **分组交换网的时延、丢包和吞吐量**



互联网可以看成是一种基础设施，该基础设施为运行在端系统上的分布式应用提供服务。我们希望在计算机网络中任意两个端系统之间传递数据都不会造成数据丢失，这是一个极高的目标，实践中难以达到。所以，在实践中必须要限制端系统之间的吞吐量用来控制数据丢失。如果在端系统之间引入时延，也不能保证不会丢失分组问题。所以我们从时延、丢包和吞吐量三个层面来看一下计算机网络。



#### **分组交换中的时延**



计算机网络中的分组从一台主机（源）出发，经过一系列路由器传输，在另一个端系统中结束它的历程。在这整个传输历程中，分组会涉及到四种最主要的时延：节点处理时延(nodal processing delay)、排队时延(queuing delay)、传输时延(transmission delay)和传播时延(propagation delay)。这四种时延加起来就是节点总时延(total nodal delay)。



如果用 dproc dqueue dtrans dpop 分别表示处理时延、排队时延、传输时延和传播时延，则节点的总时延由以下公式决定: dnodal = dproc + dqueue + dtrans + dpop。



#### **时延的类型**



下面是一副典型的时延分布图，让我们从图中进行分析一下不同的时延类型。

![image-20231009224115871](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009224115871.png)

分组由端系统经过通信链路传输到路由器 A，**路由器 A 检查分组头部以映射出适当的传输链路，并将分组送入该链路。**仅当该链路没有其他分组正在传输并且没有其他分组排在该该分组前面时，才能在这条链路上自由的传输该分组。如果该链路当前繁忙或者已经有其他分组排在该分组前面时，新到达的分组将会加入排队。下面我们分开讨论一下这四种时延。



**节点处理时延**

节点处理时延分为两部分，第一部分是路由器会检查分组的首部信息；第二部分是决定将分组传输到哪条通信链路所需要的时间。一般高速网络的节点处理时延都在微秒级和更低的数量级。在这种处理时延完成后，分组会发往路由器的转发队列中。



**排队时延**

在队列排队转发过程中，分组需要在队列中等待发送，分组在等待发送过程中消耗的时间被称为排队时延。排队时延的长短取决于先于该分组到达正在队列中排队的分组数量。如果该队列是空的，并且当前没有正在传输的分组，那么该分组的排队时延就是 0。如果处于网络高发时段，那么链路中传输的分组比较多，那么分组的排队时延将延长。实际的排队时延也可以到达微秒级。



**传输时延**

队列是路由器所用的主要的数据结构。队列的特征就是先进先出，先进食堂的先打饭。**传输时延是理论情况下单位时间内的传输比特所消耗的时间。**比如分组的长度是 L 比特，R 表示从路由器 A 到路由器 B 的传输速率。那么传输时延就是 L / R 。这是将所有分组推向该链路所需要的时间。正是情况下传输时延通常也在毫秒到微秒级。



**传播时延**

**从链路的起点到路由器 B 传播所需要的时间就是传播时延。**该比特以该链路的传播速率传播。该传播速率取决于链路的物理介质(双绞线、同轴电缆、光纤)。如果用公式来计算一下的话，该传播时延等于两台路由器之间的距离 / 传播速率。**即传播时延是 d/s** ，其中 d 是路由器 A 和 路由器 B 之间的距离，s 是该链路的传播速率。



#### **传输时延和传播时延的比较**



计算机网络中的传输时延和传播时延有时候难以区分，在这里解释一下，传输时延是路由器推出分组所需要的时间，它是分组长度和链路传输速率的函数，而与两台路由器之间的距离无关。而传播时延是一个比特从一台路由器传播到另一台路由器所需要的时间，它是两台路由器之间距离的倒数，而与分组长度和链路传输速率无关。从公式也可以看出来，传输时延是 L/R，也就是分组的长度 / 路由器之间传输速率。传播时延的公式是 d/s，也就是路由器之间的距离 / 传播速率。



#### **排队时延**



在这四种时延中，人们最感兴趣的时延或许就是排队时延了 dqueue。与其他三种时延（dproc、dtrans、dpop）不同的是，排队时延对不同的分组可能是不同的。例如，如果 10 个分组同时到达某个队列，第一个到达队列的分组没有排队时延，而最后到达的分组却要经受最大的排队时延（需要等待其他九个时延被传输）。



那么如何描述排队时延呢？或许可以从三个方面来考虑：**流量到达队列的速率、链路的传输速率和到达流量的性质**。即流量是周期性到达还是突发性到达，如果用 a 表示分组到达队列的平均速率（ a 的单位是分组/秒，即 pkt/s）前面说过 R 表示的是传输速率，所以能够从队列中推出比特的速率（以 bps 即 b/s 位单位）。假设所有的分组都是由 L 比特组成的，那么比特到达队列的平均速率是 La bps。那么比率 La/R 被称为*流量强度(traffic intensity)*，如果 La/R > 1，则比特到达队列的平均速率超过从队列传输出去的速率，这种情况下队列趋向于无限增加。所以，**设计系统时流量强度不能大于1**



#### **丢包**



我们在上述的讨论过程中描绘了一个公式那就是 La/R 不能大于1，如果 La/R 大于1，那么到达的排队将会无穷大，而且路由器中的排队队列所容纳的分组是有限的，所以等到路由器队列堆满后，新到达的分组就无法被容纳，导致路由器丢弃该分组，即分组会丢失。



#### **计算机网络中的吞吐量**



除了丢包和时延外，衡量计算机另一个至关重要的性能测度是端到端的吞吐量。假如从主机 A 向主机 B 传送一个大文件，那么在任何时刻主机 B 接收到该文件的速率就是瞬时吞吐量(instantaneous throughput)。如果该文件由 F 比特组成，主机 B 接收到所有 F 比特用去 T 秒，则文件的传送平均吞吐量(average throughput)是 F / T bps。



### **单播、广播、多播和任播**



在网络通信中，可以根据目标地址的数量对通信进行分类，可以分为 **单播、广播、多播和任播**。



#### **单播(Unicast)**



单播最大的特点就是 1 对 1，早期的固定电话就是单播的一个例子，单播示意图如下。

![image-20231009225155846](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009225155846.png)

#### **广播(Broadcast)**



我们一般小时候经常会跳广播体操，这就是广播的一个事例，主机和与他连接的所有端系统相连，主机将信号发送给所有的端系统。



![image-20231009225213129](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009225213129.png)

#### **多播(Multicast)**



多播与广播很类似，也是将消息发送给多个接收主机，不同之处在于多播需要限定在某一组主机作为接收端。



![image-20231009225308953](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009225308953.png)

#### **任播(Anycast)**



任播是在特定的多台主机中选出一个接收端的通信方式。虽然和多播很相似，但是行为与多播不同，任播是从许多目标机群中选出一台最符合网络条件的主机作为目标主机发送消息。然后被选中的特定主机将返回一个单播信号，然后再与目标主机进行通信。



![image-20231009225458120](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009225458120.png)



## **物理媒介**



网络的传输是需要介质的。一个比特数据包从一个端系统开始传输，经过一系列的链路和路由器，从而到达另外一个端系统。这个比特会被转发了很多次，那么这个比特经过传输的过程所跨越的媒介就被称为物理媒介(phhysical medium)，物理媒介有很多种，比如双绞铜线、同轴电缆、多模光纤榄、陆地无线电频谱和卫星无线电频谱。其实大致分为两种：引导性媒介和非引导性媒介。



### **双绞铜线**



最便宜且最常用的引导性传输媒介就是双绞铜线，多年以来，它一直应用于电话网。从电话机到本地电话交换机的

连线超过 99% 都是使用的双绞铜线，例如下面就是双绞铜线的实物图。



![image-20231009225610084](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009225610084.png)



双绞铜线由两根绝缘的铜线组成，每根大约 1cm 粗，以规则的螺旋形状排列，通常许多双绞线捆扎在一起形成电缆，并在双绞馅的外面套上保护层。一对电缆构成了一个通信链路。无屏蔽双绞线一般常用在局域网（LAN）中。



### **同轴电缆**



与双绞线类似，同轴电缆也是由两个铜导体组成，下面是实物图。



![image-20231009225647049](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009225647049.png)



借助于这种结构以及特殊的绝缘体和保护层，同轴电缆能够达到较高的传输速率，同轴电缆普遍应用在在电缆电视系统中。同轴电缆常被用户引导型共享媒介。



### **光纤**



光纤是一种细而柔软的、能够引导光脉冲的媒介，每个脉冲表示一个比特。一根光纤能够支持极高的比特率，高达数十甚至数百 Gbps。它们不受电磁干扰。光纤是一种引导型物理媒介，下面是光纤的实物图。



![image-20231009225731211](/assets/blog_res/2023-09-04-NetworkThree.assets/image-20231009225731211.png)



一般长途电话网络全面使用光纤，光纤也广泛应用于因特网的主干。



### **陆地无线电信道**



无线电信道承载电磁频谱中的信号。它不需要安装物理线路，并具有穿透墙壁、提供与移动用户的连接以及长距离承载信号的能力。



### **卫星无线电信道**



一颗卫星电信道连接地球上的两个或多个微博发射器/接收器，它们称为地面站。通信中经常使用两类卫星：同步卫星和近地卫星。