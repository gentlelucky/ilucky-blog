---
title: "周报 Vol.21 - 固执己见是一座大山"
date: 2025-02-24T16:00:00+08:00
draft: false
tags: 
  - "review"
  - "life"
  - "think"
  - "AI"
  - "RAG"
categories: 
  - "Ideas"
authors:
  - "gentlelucky"
---

## 前言

本篇是对  `2024-02-17`  到  `2025-02-23`  这周生活的记录和思考。

表弟一直未考上理想院校，和他聊天后发现，每个人是活在自己的认知里，虚心学习和保持好奇很重要；很好奇 AI 如何接入并落地，于是读了相关文章，了解了什么是「RAG」，Agent如何使用等；

## 固执己见是一座大山

![太乙](https://image.gentlelucky.com/%E5%A4%AA%E4%B9%99.png)

表弟从 2021 年考研至今，未考上理想院校。和他的沟通的过程中，发现他无法从多角度看待问题。说得最多是：

- 网上说......
- 我朋友说......

没有自己的思考和求证，坚信他人和他说的。有意思的是，我说的换个角度去思考问题他却全盘否定。

和他交谈的过程中，我一直是支持他的决定的。我想表达以下几点：① 分析一下没考上的原因 ② 思考一下是否非考研不可

### 分析没考上原因

我：你有分析过这几年没考上理想院校的原因吗？并针对问题，有解决方案？

表弟：我分析了，第一年没考上由于「杭电」人工智能专业报考人数太多了，导致录取分数线好高；第二年出题人出的题太小众了，我看的都是大众的题；第三年报考的「安大」计算机与科学专业的人数又比往年多......

我：考研其实分外因和内因的。外因是哪些我们无法掌控，这部分我们没有办法。就像你刚刚说的报考人数多了，出题小众了。内因是我们自己可以掌控的，像如何提高分数，选择一个好考的院校等。

表弟：不是的，你说提高分数，那出题人就出一些稀奇古怪的题目，没办法的。

我说：你所说的「稀奇古怪」的题目，应该有 90% 以上的人也不会的题目，我才会定义为「稀奇古怪」的题目。如果 90% 人都不会，那其实你和大家还是在同一个起跑线的。这部分题目可以忽略；和你沟通下来，你把这几次没考好都归结为「外因」，而自身没有任何问题。我觉得你应该先承认自己的「失败」，然后总结经验，再次出发！

表弟：没你说的那么简单，你去考考你就知道了！

### 是否非考研不可

我：你坚定的要考研，且是计算机专业的研究生。但现在大环境不好，也不知道未来计算机专业怎样。你有没有考虑过从事你本专业「护理」工作？

表弟：我不喜欢「护理」，当时毕业的时候有一个工作给我每月八千，我都没去。我就是不喜欢干。未来从事计算机工作，一定比护理轻松。

我：但是有可能等你研究生毕业，市场饱和了，就业可能有点困难。而且现在很多大厂都不愿意要 35 岁以上的技术人员，研究生 3 年毕业，你也快30岁了吧。

表弟：不会的，我看好AI，即使被裁员，我也能接受低薪工作。

我：那如果今年又没考上，你有什么打算呀？

表弟：我会一直坚持考下去，今年如果还没考上。我先去达内培训「C ++」，他们承诺毕业后合肥薪资 10k+，然后我就一边上班一边考，考研并不是越努力就能考得上，这里面有运气成分的，我就是再赌我运气能考上。

我：......

至此，我已经不想再沟通下去了。在沟通的过程中，我的话总被打断，他说的很多，感觉是一定要说服我，让我赞同他的做法。

恰巧的是，我当晚看到了一句话：

> 有时候，与那些固执己见、不肯改变观点的人，进行辩论是值得的。也许他永远不会让步，但你可以帮助其他人，看清他的胡说八道。
>
> 当然，你要警惕，不要给不法之徒提供表演的舞台，而且你的时间和精力是有限的。

是呀，我的时间和精力是有限的呀！

## AI 入门

> 说明：由于刚入门 AI，对于 AI 的理解和认知存在偏差，欢迎指出纠正！

### AI Agent

从代码层面对 **AI Agent** 的认知应该是 「Functions」 的概念，即：定义一个函数，当用户输入的信息命中了此函数，会把用户输入的信息作为入参回调此函数。

```json
curl https://api.openai.com/v1/chat/completions \
-H "Content-Type: application/json" \
-H "Authorization: Bearer $OPENAI_API_KEY" \
-d '{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": "What'\''s the weather like in Boston today?"
    }
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_current_weather",
        "description": "Get the current weather in a given location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g. San Francisco, CA"
            },
            "unit": {
              "type": "string",
              "enum": ["celsius", "fahrenheit"]
            }
          },
          "required": ["location"]
        }
      }
    }
  ],
  "tool_choice": "auto"
}'
```

此场景，可用于客服场景、对话场景、命令场景等；本质是 AI 大模型解析自然语言，触发对应函数，系统执行函数执行系统业务逻辑；

### RAG

通过阅读[零基础入门：如何用 RAG (检索增强生成) 打造知识库 QA 系统](https://github.com/rag-web-ui/rag-web-ui/blob/main/docs/tutorial/README.md#%E9%9B%B6%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8%E5%A6%82%E4%BD%95%E7%94%A8-rag-%E6%A3%80%E7%B4%A2%E5%A2%9E%E5%BC%BA%E7%94%9F%E6%88%90-%E6%89%93%E9%80%A0%E7%9F%A5%E8%AF%86%E5%BA%93-qa-%E7%B3%BB%E7%BB%9F)文章，了解了 RAG 在实际场景中的运用。

RAG 的典型应用场景

- 企业知识库问答：帮助企业构建对内员工知识库或对外客户问答系统。
- 法律法规、论文等参考场景：需要给出权威来源或证据的回答。
- 任何需要"带有引用信息"的回答场景。

## 有趣的事与物

#### 收藏

- [Your One-Stop Publication Workbench | Zettlr]([www.zettlr.com...](https://www.zettlr.com/))
- [usebruno.com]([www.usebruno.com...](https://www.usebruno.com/))
- [减小 Docker 日志大小：日志管理实用指南 --- Reducing Docker Logs Size: A Practical Guide to Log Management]([linuxiac.com...](https://linuxiac.com/reducing-docker-logs-file-size/))

#### 音乐

- [【4K60FPS】邓紫棋《唯一》超好听神曲！你真的懂唯⼀的定义]([www.bilibili.com...](https://www.bilibili.com/video/av114029989267538))

#### 视频

- [“后羿为什么不射工作日”]([www.bilibili.com...](https://www.bilibili.com/video/av114030140199908))

#### 影视

- [三叉戟2]([movie.douban.com...](http://movie.douban.com/subject/35633775/))
