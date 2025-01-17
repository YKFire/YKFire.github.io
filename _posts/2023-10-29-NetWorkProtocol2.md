---
title: 计算机网络传输层协议(二)
date: 2023-10-29 15:36:00 +0800
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



# **传输层协议**

这是计算机网络传输层的第二篇。

## **UDP**



UDP 的全称是**用户数据报协议(UDP，User Datagram Protocol)**，UDP 为应用程序提供了一种无需建立连接就可以封装并发送 IP 数据包的方法。如果应用程序开发人员选择的是 UDP 而不是 TCP 的话，那么该应用程序相当于就是和 IP 直接打交道的。



从应用程序传递过来的数据，会附加上多路复用/多路分解的源和目的端口号字段，以及其他字段，然后将形成的报文传递给网络层，网络层将运输层报文段封装到 IP 数据报中，再交付给目标主机。



### **UDP 特点**



UDP 协议一般作为流媒体应用、语音交流、视频会议所使用的传输层协议，我们大家都熟知的 DNS 协议底层也使用了 UDP 协议，这些应用或协议之所以选择 UDP 主要是因为以下这几点



1. 速度快，采用 UDP 协议时，只要应用进程将数据传给 UDP，UDP 就会将此数据打包进 UDP 报文段并立刻传递给网络层，然后 TCP 有拥塞控制的功能，它会在发送前判断互联网的拥堵情况，如果互联网极度阻塞，那么就会抑制 TCP 的发送方。使用 UDP 的目的就是希望实时性。



1. 无须建立连接，TCP 在数据传输之前需要经过三次握手的操作，而 UDP 则无须任何准备即可进行数据传输。因此 UDP 没有建立连接的时延。如果使用 TCP 和 UDP 来比喻开发人员：TCP 就是那种凡事都要设计好，没设计不会进行开发的工程师，需要把一切因素考虑在内后再开干！所以非常靠谱；而 UDP 就是那种上来直接干干干，接到项目需求马上就开干，也不管设计，也不管技术选型，就是干，这种开发人员非常不靠谱，但是适合快速迭代开发，因为可以马上上手！



1. 无连接状态，TCP 需要在端系统中维护连接状态，连接状态包括接收和发送缓存、拥塞控制参数以及序号和确认号的参数，在 UDP 中没有这些参数，也没有发送缓存和接受缓存。因此，某些专门用于某种特定应用的服务器当应用程序运行在 UDP 上，一般能支持更多的活跃用户。



1. 分组首部开销小，每个 TCP 报文段都有 20 字节的首部开销，而 UDP 仅仅只有 8 字节的开销。



这里需要注意一点，并不是所有使用 UDP 协议的应用层都是不可靠的，应用程序可以自己实现可靠的数据传输，通过增加确认和重传机制。所以使用 UDP 协议最大的特点就是速度快。



### **UDP 报文结构**



下面来一起看一下 UDP 的报文结构，每个 UDP 报文分为 UDP 报头和 UDP 数据区两部分。报头由 4 个 16 位长（2 字节）字段组成，分别说明该报文的源端口、目的端口、报文长度和校验值。



![image-20231118154522421](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118154522421.png)



1. 源端口号(Source Port) :这个字段占据 UDP 报文头的前 16 位，通常包含发送数据报的应用程序所使用的 UDP 端口。接收端的应用程序利用这个字段的值作为发送响应的目的地址。这个字段是可选项，有时不会设置源端口号。没有源端口号就默认为 0 ，通常用于不需要返回消息的通信中。



1. 目标端口号(Destination Port): 表示接收端端口，字段长为 16 位。



1. 长度(Length): 该字段占据 16 位，表示 UDP 数据报长度，包含 UDP 报文头和 UDP 数据长度。因为 UDP 报文头长度是 8 个字节，所以这个值最小为 8，最大长度为 65535 字节。



1. 校验和(Checksum)：UDP 使用校验和来保证数据安全性，UDP 的校验和也提供了差错检测功能，差错检测用于校验报文段从源到目标主机的过程中，数据的完整性是否发生了改变。发送方的 UDP 对报文段中的 16 比特字的和进行反码运算，求和时遇到的位溢出都会被忽略，比如下面这个例子，三个 16 比特的数字进行相加

![image-20231118154701200](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118154701200.png)

这些16比特的前两个和是

![image-20231118154725931](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118154725931.png)

然后再将上面的结果和第三个 16 比特的数进行相加

![image-20231118154736672](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118154736672.png)

最后一次相加的位会进行溢出，溢出位 1 要被舍弃，然后进行反码运算，反码运算就是将所有的 1 变为 0 ，0 变为 1。因此 **1000 0100 1001 0101** 的反码就是 **0111 1011 0110 1010**，这就是校验和，如果在接收方，数据没有出现差错，那么全部的 4 个 16 比特的数值进行运算，同时也包括校验和，如果最后结果的值不是 1111 1111 1111 1111 的话，那么就表示传输过程中的数据出现了差错。




下面来想一个问题，为什么 UDP 会提供差错检测的功能？



这其实是一种端到端的设计原则，这个原则说的是**要让传输中各种错误发生的概率降低到一个可以接受的水平**。UDP 不可靠的原因是它虽然提供差错检测的功能，但是**对于差错没有恢复能力更不会有重传机制**。



## **TCP**


UDP 是一种没有复杂的控制，提供无连接通信服务的一种协议，换句话说，它将部分控制部分交给应用程序去处理，自己只提供作为传输层协议最基本的功能。



而与 UDP 不同的是，同样作为传输层协议，TCP 协议要比 UDP 的功能多很多。



TCP 的全称是 Transmission Control Protocol，它被称为是一种面向连接的协议，这是因为一个应用程序开始向另一个应用程序发送数据之前，这两个进程必须先进行握手，握手是一个逻辑连接，并不是两个主机之间进行真实的握手。

![image-20231118154946959](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118154946959.png)

这个连接是指各种设备、线路或者网络中进行通信的两个应用程序为了相互传递消息而专有的、虚拟的通信链路，也叫做虚拟电路。



一旦主机 A 和主机 B 建立了连接，那么进行通信的应用程序只使用这个虚拟的通信线路发送和接收数据就可以保证数据的传输，TCP 协议负责控制连接的建立、断开、保持等工作。



TCP 连接是**全双工服务(full-duplex service)** 的，全双工是什么意思？全双工指的是主机 A 与另外一个主机 B 存在一条 TCP 连接，那么应用程数据就可以从主机 B 流向主机 A 的同时，也从主机 A 流向主机 B。



一条 TCP 连接只能是**点对点(point-to-point)**的，那么所谓的多播，即一个主机对多个接收方发送消息的情况是不存在的，TCP 连接只能连接两个一对主机。



一旦 TCP 连接建立后，主机之间就可以相互发送数据了，客户进程通过套接字发送数据。数据一旦通过套接字后，它就由客户中运行的 TCP 协议所控制。



TCP 会将数据临时存储到连接的**发送缓存(send buffer)**中，这个 send buffer 是三次握手之间设置的缓存之一，然后 TCP 在合适的时间将发送缓存中的数据发送到目标主机的接收缓存中，实际上，每一端都会有发送缓存和接收缓存，如下图所示

![image-20231118155116471](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118155116471.png)

机之间的发送是以**报文段(segment)**进行的，那么什么是 Segement 呢？



TCP 会将要传输的数据流分为多个块，然后向每个块中添加 TCP 标头，这样就形成了一个 TCP 段也就是报文段。每一个报文段可以传输的长度是有限的，不能超过**最大数据长度(Maximum Segment Size)**MSS。在报文段向下传输的过程中，会经过链路层，链路层有一个 **Maximum Transmission Unit**，最大传输单元 MTU， 即数据链路层上所能通过最大数据包的大小，最大传输单元通常与通信接口有关。



因为计算机网络是分层考虑的，这个很重要，不同层的称呼不一样，对于传输层来说，称为报文段而对网络层来说就叫做 IP 数据包，所以，**MTU 可以认为是网络层能够传输的最大 IP 数据包，而 MSS（Maximum segment size）可以认为是传输层的概念，也就是 TCP 数据包每次能够传输的最大量**。



### **TCP 报文段结构**



在简单聊了聊 TCP 连接后，下面我们就来聊一下 TCP 的报文段结构，如下图所示

![image-20231118155301275](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118155301275.png)

TCP 报文段结构相比 UDP 报文结构多了很多内容。但是前两个 32 比特的字段是一样的。它们是源端口号和目标端口号，我们知道，这两个字段是用于多路复用和多路分解的。另外，和 UDP 一样，TCP 也包含校验和，除此之外，TCP 报文段首部还有下面这些

1. 32 比特的**序号字段(sequence number field)** 和 32 比特的**确认号字段(acknowledgment number field)**。这些字段被 TCP 发送方和接收方用来实现可靠的数据传输。
2. 4 比特的**首部字段长度字段(header length field)**，这个字段指示了以 32 比特的字为单位的 TCP 首部长度。TCP 首部的长度是可变的，但是通常情况下，选项字段为空，所以 TCP 首部字段的长度是 20 字节。
3. 16 比特的**接受窗口字段(receive window field)**，这个字段用于流量控制。它用于指示接收方能够/愿意接受的字节数量
4. 可变的**选项字段(options field)**，这个字段用于发送方和接收方协商最大报文长度，也就是 MSS 时使用
5. 6 比特的**标志字段(flag field)**， ACK 标志用于指示确认字段中的值是有效的，这个报文段包括一个对已被成功接收报文段的确认；RST、SYN、FIN 标志用于连接的建立和关闭；CWR 和 ECE 用于拥塞控制；PSH 标志用于表示立刻将数据交给上层处理；URG 标志用来表示数据中存在需要被上层处理的 *紧急* 数据。紧急数据最后一个字节由 16 比特的**紧急数据指针字段(urgeent data pointer field)**指出。一般情况下，PSH 和 URG 并没有使用。



TCP 的各种功能和特点都是通过 TCP 报文结构来体现的，在聊完 TCP 报文结构之后，我们下面就来聊一下 TCP 有哪些功能及其特点了。



### **序号、确认号实现传输可靠性**



TCP 报文段首部中两个最重要的字段就是序号和确认号，这两个字段是 TCP 实现可靠性的基础，那么你肯定好奇如何实现可靠性呢？要了解这一点，首先我们得先知道这两个字段里面存了哪些内容吧？



**一个报文段的序号就是数据流的字节编号** 。因为 TCP 会把数据流分割成为一段一段的字节流，因为字节流本身是有序的，所以每一段的字节编号就是标示是哪一段的字节流。比如，主机 A 要给主机 B 发送一条数据。数据经过应用层产生后会有一串数据流，数据流会经过 TCP 分割，分割的依据就是 MSS，假设数据是 10000 字节，MSS 是 2000 字节，那么 TCP 就会把数据拆分成 0 - 1999 , 2000 - 3999 的段，依次类推。



所以，第一个数据 0 - 1999 的首字节编号就是 0 ，2000 - 3999 的首字节编号就是 2000 。。。。。。



然后，每个序号都会被填入 TCP 报文段首部的序号字段中。



![image-20231118160012569](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118160012569.png)

至于确认号的话，会比序号要稍微麻烦一些。这里我们先拓展下几种通信模型。



1. 单工通信：单工通信只支持数据在一个方向上传输；在同一时间只有一方能接受或发送信息，不能实现双向通信，比如广播、电视等。



1. 双工通信：由两个或者多个发送方同时在两个方向上通信。双工通信模型有两种：*全双工(FDX)和半双工(HDX)*
2. 半双工：在半双工系统中，连接双方可以进行通信，但不能同时通信，比如对讲机，只有把按钮按住的人才能够讲话，只有一个人讲完话后另外一个人才能讲话。
3. 全双工：在全双工系统中，连接双方可以同时进行通信，一个最常见的例子就是电话通信。全双工通信是两个单工通信方式的结合，它要求发送设备和接收设备都有独立的接收和发送能力。



单工、半双工、全双工通信如下图所示



![image-20231118160107622](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118160107622.png)

TCP 是一种全双工的通信协议，因此主机 A 在向主机 B 发送消息的过程中，也在接受来自主机 B 的数据。**主机 A 填充进报文段的确认号是期望从主机 B 收到的下一字节的序号。**稍微有点绕，我们来举个例子看一下。比如主机 A 收到了来自主机 B 发送的编号为 0 - 999 字节的报文段，这个报文段会写入序号中，随后主机 A 期望能够从主机 B 收到 1000 - 剩下的报文段，因此，主机 A 发送到主机 B 的报文段中，它的确认号就是 1000 。



#### **累积确认**



这里再举出一个例子，比如主机 A 在发送 0 - 999 报文段后，期望能够接受到 1000 之后的报文段，但是主机 B 却给主机 A 发送了一个 1500 之后的报文段，那么主机 A 是否还会继续进行等待呢？



答案是会的，因为 TCP 只会确认流中至第一个丢失字节为止的字节，因为 1500 虽然属于 1000 之后的字节，但是主机 B 没有给主机 A 发送 1000 - 1499 之间的字节，所以主机 A 会继续等待。



在了解完序号和确认号之后，我们下面来聊一下 TCP 的发送过程。下面是一个正常的发送过程



![image-20231118164149494](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118164149494.png)

主机 A 向主机 B 发送了两段报文，第一段报文是 0 - 999 ，主机 B 接收之后会发送确认应答报文，该报文中包含对主机 A 发送 0 - 999 报文段的确认号，应答报文到达主机 A 之后，经过一段时间主机 A 会发送 1000 - 1999 这段报文，主机 B 对其进行确认后再发送应答报文。

通过上图可以看到，每次主机 A 发送完报文段之后，主机 B 都会发送一个应答报文，主机 A 才会发送接下来的报文段，那么这个应答报文是啥呢？实际上，TCP 就是通过确认应答(ACK) 来实现可靠的数据传输，当主机 A将数据发出之后会等待主机 B 的响应。如果有确认应答(ACK)，说明数据已经成功到达。反之，则数据很可能会丢失。



如下图所示，如果在一定时间内主机 A 没有等到确认应答，则认为主机 B 发送的报文段已经丢失，并进行重发。



![image-20231118164248550](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118164248550.png)



主机 A 给主机 B 的响应可能由于网络抖动等原因无法到达，那么在经过特定的时间间隔后，主机 A 将重新发送报文段。



主机 A 没有收到主机 B 的应答报文，可能是因为报文段在主机 B 在发送给主机 A 的过程中丢失。



![image-20231118164335222](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118164335222.png)



如上图所示，由主机 B 返回的确认应答，由于网络拥堵等原因在传送的过程中丢失，并没有到达主机 A。主机 A 会等待一段时间，如果在这段时间内主机 A 仍没有等到主机 B 的响应，那么主机 A 会重新发送报文段。



要辩证的看待问题，如果主机 A 没有收到应答报文，不一定是主机 A 发送的报文段丢失，还有可能是主机 B 发送的应答报文丢失，还可能是主机 B 没有发送应答报文，所以没有收到报文的情况有很多种。



那么现在就存在一个问题，如果主机 A 给主机 B 发送了一个报文段后，主机 B 接受到报文段发送响应，此刻由于网络原因，这个报文段并未到达，等到一段时间后主机 A 重新发送报文段，然后此时主机 B 发送的响应在主机 A 第二次发送后失序到达主机 A，那么主机 A 应该如何处理呢？



![image-20231118164417899](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118164417899.png)

TCP RFC 并未为此做任何规定，也就是说，我们可以自己决定如何处理失序到达的报文段。一般处理方式有两种



1. 接收方立刻丢弃失序的报文段。
2. 接收方接受时许到达的报文段，并等待后续的报文段。



一般来说通常采取的做法是第二种。



### **传输控制**



#### **利用窗口控制提高速度**



前面我们介绍了 TCP 是以数据段的形式进行发送，如果经过一段时间内主机 A 等不到主机 B 的响应，主机 A 就会重新发送报文段，接受到主机 B 的响应，再会继续发送后面的报文段，我们现在看到，这一问一答的方式还存在许多意外条件，比如响应未收到、等待响应等，那么对崇尚性能的互联网来说，这种方式的性能应该不会很高。

![image-20231118164553763](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118164553763.png)

那么如何提升性能呢？



为了解决这个问题，TCP 引入了**窗口**这个概念，这个窗口大家可以把它理解为发送期，就是说在这个窗口（发送期）中，通信双方可以任意发送数据，也就是说引入窗口后，从之前单次发送变成了一段时间内的多次报文发送。所以，即使在往返时间较长、频次很多的情况下，它也能控制网络性能的下降，如下图所示

![image-20231118164832089](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118164832089.png)

我们之前每次请求发送都是以报文段的形式进行的，引入窗口后，每次请求都可以发送多个报文段，也就是说一个窗口可以发送多个报文段。窗口大小就是指无需等待确认应答就可以继续发送报文段的最大值。



在这个窗口机制中，大量使用了缓冲区的实现方式，通过对多个段同时进行确认应答的功能。



如下图所示，发送报文段中高亮部分即是我们提到的窗口，在窗口内，即是没有收到确认应答也可以把请求发送出去。不过，在整个窗口的确认应答没有到达之前，如果部分报文段丢失，那么主机 A 将仍会重传。为此，主机 A 需要设置缓存来保留这些需要重传的报文段，直到收到他们的确认应答。

![image-20231118164933422](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118164933422.png)

在滑动窗口以外的部分是尚未发送的报文段和已经接受到的报文段，如果报文段已经收到确认则不可进行重发，此时报文段就可以从缓冲区中清除。



在收到确认的情况下，会将窗口滑动到确认应答中确认号的位置，如上图所示，这样可以顺序的将多个段同时发送，用以提高通信性能，这种窗口也叫做 滑动窗口(Sliding window)。



#### **窗口控制和重发**



报文段的发送和接收，必然伴随着报文段的丢失和重发，窗口也是同样如此，如果在窗口中报文段发送过程中出现丢失怎么办？



首先我们先考虑确认应答没有返回的情况。在这种情况下，主机 A 发送的报文段到达主机 B，是不需要再进行重发的。这和单个报文段的发送不一样，如果发送单个报文段，即使确认应答没有返回，也要进行重发。



![image-20231118165053955](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118165053955.png)

**窗口在一定程度上比较大时，即使有少部分确认应答的丢失，也不会重新发送报文段。**



我们知道，如果在某个情况下由于发送的报文段丢失，导致接受主机未收到请求，或者主机返回的响应未到达客户端的话，会经过一段时间重传报文。那么在使用窗口的情况下，报文段丢失会怎么样呢？



如下图所示，报文段 0 - 999 丢失后，但是主机 A 并不会等待，主机 A 会继续发送余下的报文段，主机 B 发送的确认应答却一直是 1000，同一个确认号的应答报文会被持续不断的返回，如果发送端主机在连续 3 次收到同一个确认应答后，就会将其所对应的数据重发，这种机制要比之前提到的超时重发更加高效，这种机制也被称为高速重发控制。这种重发的确认应答也被称为**冗余 ACK(响应)**。

![image-20231118165253570](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118165253570.png)

主机 B 在没有接收到自己期望序列号的报文段时，会对之前收到的数据进行确认应答。发送端则一旦收到某个确认应答后，又连续三次收到同样的确认应答，那么就会认为报文段已经丢失。需要进行重发。使用这种机制可以提供更为快速的重发服务。



### **流量控制**



我们知道，在每个 TCP 连接的一侧主机都会有一个 socket 缓冲区，缓冲区会为每个连接设置接收缓存和发送缓存，当 TCP 建立连接后，从应用程序产生的数据就会到达接收方的接收缓冲区中，接收方的应用程序并不一定会马上读取缓冲区的数据，它需要等待操作系统分配时间片。如果此时发送方的应用程序产生数据过快，而接收方读取接受缓冲区的数据相对较慢的话，那么接收方中缓冲区的数据将会溢出，导致数据丢失。



但是还好，TCP 有**流量控制服务(flow-control service)**机制用于消除缓冲区溢出的情况。流量控制是一个速度匹配服务，即发送方的发送速率与接受方应用程序的读取速率相匹配。



TCP 通过使用一个**接收窗口(receive window)** 的变量来提供流量控制。接收窗口会给发送方一个指示到底还有多少可用的缓存空间。发送端会根据接收端的实际接受能力来控制发送的数据量。



接收端向发送端通知自己可以接收数据量的大小，发送端会发送不超过这个限度的数据，这个大小限度就是窗口大小，还记得 TCP 的首部么，有一个接收窗口，我们上面聊的时候说这个字段用于流量控制。它用于指示接收方能够接受的字节数量。



那么如何实时知道接收方能够接收的数据量大小呢？



发送端主机会定期发送一个**窗口探测包**，这个包用于探测接收端主机是否还能够接受数据，当接收端的缓冲区一旦面临数据溢出的风险时，窗口大小的值也随之被设置为一个更小的值通知发送端，从而控制数据发送量。

下面是一个流量控制示意图

![image-20231118165451690](/assets/blog_res/2023-10-29-NetWorkProtocol2.assets/image-20231118165451690.png)

发送端主机根据接收端主机的窗口大小进行流量控制。由此也可以防止发送端主机一次发送过大数据导致接收端主机无法处理。



如上图所示，当主机 B 收到报文段 2000 - 2999 之后缓冲区已满，不得不暂时停止接收数据。然后主机 A 发送窗口探测包，窗口探测包非常小仅仅一个字节。然后主机 B 更新缓冲区接收窗口大小并发送窗口更新通知给主机 A，然后主机 A 再继续发送报文段。



在上面的发送过程中，窗口更新通知可能会丢失，一旦丢失发送端就不会发送数据，所以窗口探测包会随机发送，以避免这种情况发生。
