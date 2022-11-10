---
title: Git&GitHub使用
date: 2022-09-09 18:19:00 +0800
categories: [实用工具]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---

# Git&GitHub使用 

本文将带你快速入门Git&GitHub

Git是一个版本控制软件，而GitHub是基于Git这一个版本控制软件的远程仓库网站。

## 一、Git使用

### 1、Git的历史和现状

>Git 是 Linux 作者 Linus 的另一个作品。2002 年他还在使用 Bitkeeper 作为 Linux 内核的版本管理，但因为它是 Copyright 有版权的软件备受质疑，然后 Andrew Tridgell 对 Bitkeeper 进行逆向工程，导致 BitMover 要回收 Linux 开发者的 Bitkeeper 的免费使用权，Linus 一怒之下花了 10 天写出了 Git。
>
>名字的意思是：egotistical bastard
>
>如今 Git 已经成为绝大多数开发者的选择， Tom Preston-Werner、Chris Wanstrath 和 PJ Hyett 在 2007 年 10 月推出的 Github 已经成为了全球最大的开发者网站。

### 2、集中式版本管理和分布式版本管理的区别

>Git 和 SVN 是从设计理念上就不一样的版本工具，SVN 将代码进行中心化管理，拥有更好的稳定性和安全性，但是去中心化的 Git 却是从 Linux 操作系统的开发需求而来，更加适合多人协作的开源项目，可以以任何一个点为 remote 将他的代码与本地代码合并，随着时间发展，还衍生出了更多强大功能和一整套操纵流程，让它也可以适应了商业软件的开发。

![集中式版本管理和分布式版本管理的区别](/assets/blog_res/2022-09-09-Git&GitHub.assets/image-20220909200518394.png)

### 3、Git基础命令

#### 1.Git 按照场景可以分为以下场景

1. Workspace：**当前工作区**，修改的的最初状态。
2. Staging：修改后，添加到准备提交的缓存状态，**即暂存区**。
3. Local repository：**本地的代码仓库**，只对自己的代码生效。这也是和 svn 区别之一，svn commit 之后就直接提交到远程服务器了，git commit 之后只是到本地代码库。
4. Remote repository：**远程代码库**，将自己的本地代码库同步到远程代码库上，这样可以供别的开发者分享自己的成果；典型的远程仓库有：GitHub、Gitee....

#### 2.常用的Git命令

- git add：添加到暂存区
- git commit：提交到本地仓库
- git push：提交到远程仓库
- git fetch：将项目拉取到本地仓库
- git clone：添加到本地工作区

![Git基础命令](/assets/blog_res/2022-09-09-Git&GitHub.assets/image-20220909200546737.png)



## 二、GitHub使用

### 1、GitHub简介

> GitHub 是一个面向开源及私有软件项目的托管平台，因为只支持 Git 作为唯一的版本库格式进行托管，故名 GitHub。GitHub 于 2008 年 4 月 10 日正式上线，除了 Git 代码仓库托管及基本的 Web 管理界面以外，还提供了订阅、讨论组、文本渲染、在线文件编辑器、协作图谱（报表）、代码片段分享（Gist）等功能。目前，在 GitHub 上托管的版本数量非常之多，其中不乏知名开源项目 Ruby on Rails、jQuery、python 等。

### 2、GitHub的软收藏与硬收藏

- 硬收藏：git clone(远程拉取到文件夹) 或者 Download zip(下载成本地压缩包)
- 软收藏： star:收藏   fork:复制到自己的本地仓库

![GitHub的软收藏与硬收藏](/assets/blog_res/2022-09-09-Git&GitHub.assets/image-20220909200208630.png)

### 3、GitHub基本功能板块

1. Pull reques：是一种通知机制。 你修改了他人的代码，将你的修改通知原来的作者，希望他合并你的修改，让项目进一步完善
2. Issues：“评论区”，你可以理解为在一个项目下面提出自己的想法告诉作者哪里可以改进
3. Explore："发现页"，你可以在这一模块探索有趣的项目，其中**Trending模块**可以让你通过调整**国家、编程语言、最近时间**来获取最近最热门的项目

![GitHub基本功能板块](/assets/blog_res/2022-09-09-Git&GitHub.assets/image-20220909195935778.png)

### 4、GitHub关键词搜索

- 搜索前缀： **awesome** xxxx(技术名称) 用于搜索该技术的权威资料
- 搜索后缀： xxxx(技术名称)  sample   用于搜索该技术的相关项目
- 搜索后缀： xxxx(技术名称)  start  用于搜索该技术的初始化配置项目

### 5、优秀资源推荐

- HelloGitHub： 分享 GitHub 上有趣、入门级的开源项目。**每月 28 号**以月刊的形式[更新发布](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MzYyNzQ0MQ==&action=getalbum&album_id=1331197538447310849#wechat_redirect)，内容包括：**有趣、入门级的开源项目**、**开源书籍**、**实战项目**、**企业级项目**等，让你用很短时间感受到开源的魅力，爱上开源  地址：https://github.com/521xueweihan/HelloGitHub
- 阮一峰老师的科技爱好者周刊：记录每周值得分享的科技内容，周五发布。  地址：https://github.com/ruanyf/weekly





