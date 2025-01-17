---
title: 通过调试技术，我理清楚了b站视频播放很快的原理
date: 2023-07-24 13:28:00 +0800
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

## range应用 

b 站视频播放的是很快的，基本是点哪就播放到哪。

而且如果你上次看到某个位置，下次会从那个位置继续播放。

那么问题来了：如果一个很大的视频，下载下来需要很久，怎么做到点哪个位置快速播放那个位置的视频呢？

前面写过一篇 range 请求的文章，也就是不下载资源的全部内容，只下载 range 对应的范围的部分。

那视频的快速播放，是不是也是基于 range 来实现的呢？

我们先复习下 range 请求：

![image-20230820145713812](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820145713812.png)

请求的时候带上 range：

![image-20230820145728396](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820145728396.png)

服务端会返回 206 状态码，还有 Content-Range 的 header 代表当前下载的是整个资源的哪一部分：

![image-20230820145746065](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820145746065.png)

这里的 Content-Length 是当前内容的长度，而 Content-Range 里是资源总长度和当前资源的范围。

想要了解更多关于 Range 的介绍可以看这篇文章：[基于 HTTP Range 实现文件分片并发下载 | YKFire](https://ykfire.github.io/posts/range/)

那 b 站视频是不是用 Range 来实现的快速播放呢？

我们先在知乎的视频试一下：

随便打开一个视频页面，然后打开 devtools，刷新页面，拖动下进度条，可以看到确实有 206 的状态码：

![image-20230820145904634](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820145904634.png)

我们可以在搜索框输入 status-code:206 把它过滤出来：

![image-20230820145915606](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820145915606.png)

这是一种叫过滤器的技巧：

![image-20230820145927598](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820145927598.png)

可以根据 method、domain、mime-type 等过滤。

has-response-header：过滤响应包含某个 header 的请求

method：根据 GET、POST 等请求方式过滤请求

domain: 根据域名过滤

status-code：过滤响应码是 xxx 的请求，比如 404、500 等

larger-than：过滤大小超过多少的请求，比如 100k，1M

mime-type：过滤某种 mime 类型的请求，比如 png、mp4、json、html 等

resource-type：根据请求分类来过滤，比如 document 文档请求，stylesheet 样式请求、fetch 请求，xhr 请求，preflight 预检请求

cookie-name：过滤带有某个名字的 cookie 的请求

当然，这些不需要记，输入一个 - 就会提示所有的过滤器：

![image-20230820150014907](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150014907.png)

但是这个减号之后要去掉，它是非的意思。

然后点开状态码为 206 的请求看一下：

![image-20230820150111371](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150111371.png)

![image-20230820150118047](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150118047.png)

确实，这是标准的 range 请求。

我点击进度条到后面的位置，可以看到发出了新的 range 请求：

![img](/assets/blog_res/2023-07-24-rangeApplication.assets/lkLloO0q8IYvRq1oWPURw710b2gR.gif)

那这些 range 请求有什么关系呢？

我们需要分析下 Content-Range，但是一个个点开看不直观。

这时候可以自定义显示的列：

右键单击列名，可以勾选展示的 header，不过这里面没有我们想要的 header，需要自定义：

![image-20230820150230812](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150230812.png)

点击 Manage Header Columns  添加自定义的 header，输入 Content-Range:

![image-20230820150302702](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150302702.png)

这时候就可以直观的看出这些 range 请求的范围之间的关系：

![image-20230820150313710](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150313710.png)

点击 Content-Range 这一列，升序排列。

我们刷新下页面，从头来试一下：

随着视频的播放，你会看到一个个 range 请求发出：

![image-20230820150329186](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150329186.png)

这些 range 请求是能连起来的，也就是说边播边下载后面的部分。

视频进度条这里的灰条也在更新：

![image-20230820150343058](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150343058.png)

当你直接点击后面的进度条：

![image-20230820150404024](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150404024.png)

观察下 range，是不是新下载的片段和前面不连续了？

也就是说会根据进度来计算出 range，再去请求。

那这个 range 是完全随意的么？

并不是。

我们当前点击的是 15:22 的位置：

![image-20230820150423990](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150423990.png)

我刷新下页面，点击 15:31 的位置：

![image-20230820150454367](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150454367.png)

如果是任意的 range，下载的部分应该和之前的不同吧。

但是你观察下两次的 range，都是 2097152-3145727

也就是说，视频分成多少段是提前就确定的，你点击进度条的时候，会计算出在哪个 range，然后下载对应 range 的视频片段来播放。

那有了这些视频片段，怎么播放呢？

浏览器有一个 SourceBuffer 的 api，我们在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/SourceBuffer) 看一下：

![image-20230820150602036](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150602036.png)

大概是这样用的：

![image-20230820150623919](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150623919.png)

也就是说，可以一部分一部分的下载视频片段，然后 append 上去。

拖动进度条的时候，可以把之前的部分删掉，再 append 新的：

![image-20230820150636390](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150636390.png)

我们验证下，搜索下代码里是否有 SourceBuffer：

按住 command + f 可以搜索请求内容：

![image-20230820150647783](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150647783.png)

可以看到搜索出 3 个结果。

在其中搜索下 SourceBuffer：

![image-20230820150707703](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150707703.png)

可以看到很多用到 SourceBuffer 的方法，基本可以确认就是基于 SourceBuffer 实现的。

也就是说，**知乎视频是通过 range 来请求部分视频片段，通过 SourceBuffer 动态播放这个片段，来实现的快速播放的目的。具体的分段是提前确定好的，会根据进度条来计算出下载哪个 range 的视频。**

那服务端是不是也要分段存储这些视频呢？

确实，有这样一种叫做 m3u8 的视频格式，它的存储就是一个个片段 ts 文件来存储的，这样就可以一部分一部分下载。

![image-20230820150731383](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150731383.png)

不过知乎没用这种格式，还是 mp4 存储的，这种就需要根据 range 来读取部分文件内容来返回了：

![image-20230820150747850](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150747850.png)

再来看看 b 站，它也是用的 range 请求的方式来下载视频片段：

![image-20230820150811015](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150811015.png)

大概 600k 一个片段：

![image-20230820150819842](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150819842.png)

下载 600k 在现在的网速下需要多久？这样播放能不快么？

相比之下，知乎大概是 1M 一个片段：

![image-20230820150837359](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150837359.png)

网速不快的时候，体验肯定是不如 b 站的。

而且 b 站用的是一种叫做 m4s 的视频格式：

![image-20230820150907534](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150907534.png)

它和 m3u8 类似，也是分段存储的，这样提前分成不同的小文件，然后 range 请求不同的片段文件，速度自然会很快。

然后再 command + f 搜索下代码，同样是用的 SourceBuffer：

![image-20230820150926135](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820150926135.png)

这样，我们就知道了为什么 b 站视频播放的那么快了：

**m4s 分段存储视频，通过 range 请求动态下载某个视频片段，然后通过 SourceBuffer 来动态播放这个片段。**

## 总结



我们分析了 b 站、知乎视频播放速度很快的原因。

结论是**通过 range 动态请求视频的某个片段，然后通过 SourceBuffer 来动态播放这个片段。**

这个 range 是提前确定好的，会根据进度条来计算下载哪个 range 的视频。

播放的时候，会边播边下载后面的 range，而调整进度的时候，也会从对应的 range 开始下载。

服务端存储这些视频片段的方式，b 站使用的 m4s，当然也可以用 m3u8，或者像知乎那样，动态读取 mp4 文件的部分内容返回。

除了结论之外，调试过程也是很重要的：

我们通过 status-code 的过滤器来过滤出了 206 状态码的请求。

![image-20230820151013732](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820151013732.png)

通过自定义列在列表中直接显示了 Content-Range：

![image-20230820151029083](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820151029083.png)

通过 command + f 搜索了响应的内容：

![image-20230820151049911](/assets/blog_res/2023-07-24-rangeApplication.assets/image-20230820151049911.png)

这篇文章就是对这些调试技巧的综合运用。

以后再看 b 站和知乎视频的时候，你会不会想起它是基于 range 来实现的分段下载和播放呢？
