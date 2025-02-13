---
title: "周报 Vol.04 - 信息输入: Follow 初体验"
date: 2024-09-22T14:45:00+08:00
draft: false
tags: 
  - "review"
  - "life"
  - "pkm"
  - "follow"
  - "tool"
  - "network"
  - "homelab"
  - "homeserver"
categories: 
  - "Ideas"
authors:
  - "gentlelucky"
---

## 前言

本篇是对  `2024-09-16`  到  `2024-09-22`  这周生活的记录和思考。

这一周用上了 [Follow](https://follow.is/)，把RSS订阅迁移到了 `follow` 上；对 `RSS` 信息的输入进行了流程梳理；重新规划了一下家庭网络架构，后期准备使用 `k8s` 把软件统一管理起来；

## RSS信息输入

### 现状

自建了 [FreshRSS](https://freshrss.org/) 聚合服务，用于订阅 `RSS` 服务。

![image-20240922160653719](https://image.gentlelucky.com/image-20240922160653719.png)

手机端、电脑端均使用 [Fluent Reader](https://hyliu.me/fluent-reader/) 应用，作为 `RSS` 的阅读器，采用 `Fever API` 方式连接自建的 [FreshRSS](https://freshrss.org/) 服务。

使用这套方案，基本满足我使用 `RSS` 信息输入的需求，但也有一些不满足我的需求：

- 没有总结文章的功能
- 当阅读了一篇不错的文章，想快速快速收藏到 `Instapaper` 应用中，不支持

### Follow

#### 介绍

[Follow](https://follow.is/) 的 slogon 是「 **Next-Gen Information Browser** 」（新一代信息浏览器）。汇聚 Web 1、2 和 3 的内容，包括文章、视频和社交媒体。它利用内置的 AI功能提供个性化推荐、摘要和翻译服务。通过 $POWER 代币，用户可以支持创作者，打造去平台化的内容消费体验。该平台旨在为用户提供个性化的信息管理和消费方式。

[Follow](https://follow.is/) 底层集成了 [RSSHub](https://docs.rsshub.app/) 应用，自然也就继承了RSSHub 的理念「万物皆可 RSS」。

#### 使用

![image-20240922164113434](https://image.gentlelucky.com/image-20240922164113434.png)

Follow 的 UI 设计真的长在我的审美点上了，以及很多交互体验上的小细节。再加上看了开发者拾一在博客「[浅谈 Follow 中的设计理念](https://innei.in/posts/design/design-concepts-in-follow-app)」中的内容，更加被他所折服。

Follow 有三块区域，分别是：

- 左侧：新增订阅、订阅源操作
- 中间：展示所订阅源发布内容的概览
- 右侧：显示具体内容（文章、图片、视频......）

Follow 对图片和视频的展示方式，也是一个很大的亮点：

![image-20240922165816454](https://image.gentlelucky.com/image-20240922165816454.png)

这种图片展示方式，可以订阅优质的数据源来提升自己的设计和审美。还可以订阅一些使人心情愉悦的图片🤗。

![image-20240922214329223](https://image.gentlelucky.com/image-20240922214329223.png)

针对于「**哔哩哔哩**」或者「**小宇宙**」，这种音视频可直接在 Follow 直接播放，这样就不用来回在「**哔哩哔哩**」和「**小宇宙**」之间切换应用了。

[Follow](https://follow.is/) 使用 `$Power` 代币可进行邀请新用户和打赏文章。欢迎关注我的 follow（https://app.follow.is/profile/@gentlelucky）来打赏我吧~🤗。

### 我的信息输入

针对于信息的收集，我主要来源是：

> 1. 突然的想法和灵感
> 2. 浏览网页时，看到引发思考的文章、值的收藏的文章
> 3. 少数派、美团技术等社区文章
> 4. 个人博客
> 5. 书籍

综上，我把信息源分为了三类：

- 闪念想法：突然的想法和灵感
- 被动信息：被动接收的信息，如各大平台推送的信息
- 主动信息：主动阅读书籍、文章等

#### 闪念想法

脑袋中突发的想法，我需要快速和方便的记录下来。所以我采用了各平台都支持的「滴答清单」应用。等方便的时候，把想法再过滤一下，如果想法有意义则收纳到 `Obsidian` 中的笔记中。

#### 被动消息

此类消息，用 `Follow` 去订阅感兴趣的信息源。每天会先大致浏览一下文章概要，如果对文章内容感兴趣则会点击收藏，等空闲下来，再精读一遍，如果文章很值得收藏，则会同步到 `Instapaper` 中。

我针对于信息源的订阅，有三个标准：

- 把控订阅质量、控制订阅数量（具体的数量要待这段时间实践后确定）
- 信息源内容一定是属于自己PARA中的某个项目
- 聚合各种领域

综上所述，`Follow` 文章流程如下：

1. 看文章标题
2. 看文章目录
3. 快速阅读
4. 收藏
5. starred to instapaper

## Home Server

### 网络规划

整体的家庭网络拓扑图如下：

![image-20240922225241994](https://image.gentlelucky.com/image-20240922225241994.png)

入户光猫改桥接模式，主路由 `iKuai` 进行拨号上网，旁路由使用`OpenWrt` 来安装插件，做一些定制化的网络需求。然后从软路由把网络分发到电脑、家庭服务器、NAS等设备上。

![image-20240922230048306](https://image.gentlelucky.com/image-20240922230048306.png)

主路由：使用 `iKuai` 进行拨号上网，动态域名、端口映射等功能设置。

![image-20240922230923884](https://image.gentlelucky.com/image-20240922230923884.png)

旁路由：使用 esir 大神构建的 `OpenWrt` 来做一些定制化的网络需求。

![image-20240922231054286](https://image.gentlelucky.com/image-20240922231054286.png)

NAS：主要用`群晖 NAS`做文件的同步和存储。

![image-20240922231425353](https://image.gentlelucky.com/image-20240922231425353.png)

家庭服务器：在 `ESXI` 上构建所需软件服务，比如：n8n、RDS......

### 服务器分类

![image-20240922231930684](https://image.gentlelucky.com/image-20240922231930684.png)



## 有趣的事与物

### 输入

虽然大部分有意思的输入会在 「[Lucky’s Footprints](https://t.me/wxluckya)」 Telegram 频道里自动同步，不过还是挑选一部分在这里列举一下。并且把 Telegram Channel 消息作为内容源搭建了一个微博客 —— 「[daily.gentlelucky.com](https://daily.gentlelucky.com/)」，可以更方便浏览了。

#### 书籍

- [我的阿勒泰](https://book.douban.com/subject/35552619/)

#### 影视

- [孤注一掷](https://movie.douban.com/subject/35267224/)

#### 文章

- [聊聊未来技术趋势](https://tw93.fun/2024-09-09/future.html)
- [环阿尔卑斯七国，三千公里的山与海](https://sspai.com/post/91374)
- [缓存：高并发读的救世主](https://www.codesky.me/archives/cache-intro.wind)
