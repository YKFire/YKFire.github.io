---
title: 完蛋！我被算法包围了
date: 2023-11-26 15:29:00 +0800
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



# **算法在日常生活的应用**

算法听上去很复杂，离我们很遥远。但是以下 10 种算法广泛用于我们的日常生活，包括互联网搜索引擎、社交网络、WiFi、手机甚至卫星。



以下是主宰我们世界的十种算法：

![image-20231210153144062](/assets/blog_res/2023-11-26-surroundedByAlgorithms.assets/image-20231210153144062.png)



## 01 排序

我们学习算法一般都是从排序算法开始的，比如冒泡排序、快排序、归并排序、堆排序等。

笔者曾经做过的一个大批量对账系统，就是在第一步使用**归并外排序**（External MergeSort）来整理数据并对齐来自不同系统的数据。很难想象如果没有排序算法，这个第一步会花费多少时间。



## 02 傅里叶变换和快速傅里叶变换

傅立叶变换用于信号在时域和频域之间的变换。傅里叶变换在医学、数据科学、物理学、声学、光学、结构力学、量子力学、数论、组合数学、概率论、统计学、信号处理、密码学、大气科学、海洋学、通讯、金融等领域都有着广泛的应用（Wikipedia）。

下图展示了傅立叶变换的原理 - 一个波形都可以用不同的正弦波来叠加合成。将复杂波形分解为不同的正弦波后，就可以做滤波处理。

![image-20231210153327876](/assets/blog_res/2023-11-26-surroundedByAlgorithms.assets/image-20231210153327876.png)



比如在图像处理中，使用离散傅立叶变换将图像变为频谱图后，可以做如下操作：

1. 去除某个特定频率的噪声
2. 减弱高频信号，降低图像清晰度和对比度，方便压缩
3. 增强高频信号，提高图像细节清晰度和对比度



## 03 Dijkstra 算法

Dijkstra 算法是由荷兰计算机科学家Edsger Dijkstra在1956年发现的算法，用于在图上两个顶点间搜索最短路径。Dijkstra 算法广泛应用于交通、寻路、规划中（Wikipedia）。



## 04 RSA 算法

RSA 算法是非对称加密算法，名称来源于三名提出者的姓氏首字母，广泛应用于网络通信、金融、军工等。“非对称”是说 RSA 算法的加密和解密使用的是不同的密钥：加密使用公钥，解密使用私钥。公钥是公开的，任何人都可以获取并用其进行加密；私钥是不公开的，只有生成密钥对的人才能持有。

我们熟悉的 HTTPS，其 TLS 握手就是用 RSA 算法实现的。由于 RSA 算法计算量较大，在握手成功后，后续的数据通信都使用基于 Session Key 的对称加密。



## 05 安全散列算法（Secure Hash Algorithm）

SHA 系列算法将文本转换为相同长度字符串，此过程不可逆。主要用于文件传输时的消息摘要、数字签名等。



## 06 质因数分解（Integer Factorization）

质因数分解是将一个正整数分解为几个质数约数的乘积的过程。这个算法应用于密码学中，比如上文提到的 RSA 算法就使用了质因数分解来计算密钥。



## 07 链接分析（Link Analysis）

链接分析，起源于对Web结构中超链接的多维分析。广泛应用于网络信息检索、数据挖掘、Web结构建模、社交网络分析等。比如在搜索引擎中，就是根据网页链接的流行度来给网页排序的。一个网页拥有的反向链接越多，就越可能是高质量网页。这样也就催生了SEO（Search Engine Optimization）这个职能，来逆向工程各种搜索引擎的排名规则。



## 08 PID 算法

PID（Proportional Integral Derivative）算法翻译过来就是比例、积分、微分控制，分别对应于图中的 3 个控制模块。这 3 种模块的输出会反馈叠加到系统的输入中，来控制系统的行为，使其稳定在一个目标值上。这个算法广泛应用于控制系统，比如温度控制，无人机姿态控制等。



## 09 数据压缩算法

我们平时浏览网页时接触到的音频、视频、文件是经过压缩的，这样可以极大地提高数据在网络上传输的速度，优化用户体验。比如用于有损图片压缩的 JPEG 算法，用于音频压缩的 MP3 算法等。



## 10 随机数生成

随机数最重要的特性是：后面生成的数与前面的数毫无关系。我们在编程中经常使用到的是伪随机数，是通过固定算法生成的。

随机数生成在密码学中很重要，常用于生成密钥。金融行业对资产风险进行场景模拟时，常使用蒙特卡洛仿真，这也需要用到随机数，以保证测试数据的统计学随机性。
