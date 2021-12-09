---
title: 《Streaming Systems》Preface 前言
thumbnail: /img/streaming-systems/cover.png
key: streaming-systems-preface
categories:
- [读书笔记, Streaming Systems]
tags:
- Streaming Systems
toc: true
---

《Streaming Systems》一书在网上得到了一致的好评与推荐，我最近也刚开始读。该书目前还没有中文翻译版本，我打算按照书中章节的顺序，对每章的内容进行相关整理，方便后续的总结与回顾。因为有很多名词可能暂时无法准确地翻译成中文，因此在整理过程中可能会出现很多中英文夹杂的情况。希望读完这本书后可以对流处理的设计、发展和存在的关键问题等方面有一个更高以及更深层次的认识。

<!--more-->

前言部分主要介绍了本书的三位作者以及他们各自撰写的章节、阅读指导、本书希望能够带给读者带来的收获以及一些可获得的资源。

一个有趣的地方是，作者特地提到他们本来要求动物书的封面上是一只机器恐龙，但是O'Reilly认为它不能很好地转化为线条艺术，虽然作者并不同意这个看法，但最后还是选择了一条鲑鱼作为妥协，也就是现在封面上所能看到的那只。以下内容是对书中部分前言内容的翻译*（由于尚未读完，因此一些内容表述可能有误，尤其是章节内容的概括部分，后续阅读完相关章节及全部内容后会修改为更加准确的表述）*：

## 阅读指导

这本书在概念上主要分为两个部分，每个部分都有四个章节，这两部分的四个章节后都各自有着一个相对独立的章节。

第一部分 The Beam Model （第一章到第四章），主要介绍了为 Google Cloud Dataflow 所开发的高层批加流数据处理模型，它后来作为 Apache Beam 这个项目捐赠给了 Apache 软件基金会。这部分由以下四个章节组成：

- 第一章，*Streaming 101*，介绍了流处理的基础概念，建立了一些术语，讨论了流处理系统的能力，区分了两种重要的时间概念（处理时间和事件时间），最后讨论了一些通用的数据处理模式。
- 第二章，回答了关于数据处理的 <font color=orange>What</font>, <font color=blue>Where</font>, <font color=green>When</font> 和 <font color=red>How</font> 这四个问题，这些问题涵盖了针对无序数据的可靠流处理中的核心概念细节。
- 第三章，*Watermarks*（水位线），深度考察了时间过程的相关指标、水位线是如何创建的以及如何在 pipeline 中传播的。最后考察了两个真实世界中水位线的实现细节。
- 第四章，*Advanced Windowing*（高级窗口），讨论一些高级的窗口和触发器概念，比如处理时间窗口，会话和自定义触发器。

在第一部分和第二部分之间，第五章介绍了 *Exactly-Once* 和 *Side Effects*，这一章中作者列举了实现端到端 exactly-once (or effectively-once) 处理语义所存在的挑战，介绍了三种不同的实现 exactly-once处理语义方法的实现细节，分别是：Apache Flink，Apache Spark 和 Google Cloud Dataflow。

第二部分，Streams and Tables（第六章到第九章），从概念上研究了从底层上思考流处理的两种方式：流和表。这部分同样包括四个章节：

- 第六章，*Streams and Tables*，介绍了流和表的基本概念，透过流和表的视角分析传统的 MapReduce 方法，并且建立了流和表的一般性理论，可以囊括Beam Model中的所有内容。
- 第七章，*The Practicalities of Persistent State*（持久化状态实践），考察两种一般类型的状态，并且分析一个实际用例，以了解一般状态管理机制的必要特征。 
- 第八章，*Streaming SQL*，研究了在关系代数和SQL的语义下流处理的含义，对比了在 Beam Model 和传统 SQL 中的固有的流和表的概念。
- 第九章，*Streaming Joins*，研究了一系列不同的join类型，分析它们在流的上下文中的行为。

最后，第十章 *The Evolution of Large-Scale Data Processing*，浏览了 MapReduce 一族数据处理系统的重要历史，考察了一些重要的贡献，这些贡献使得流处理系统演变成了今天的样子。

## 带给读者的收获

- 你可以从本书中学到的最重要的一点是流和表的理论以及它们是如何相互关联的，其他的一切都建立在此之上。从第六章开始会接触到这一话题。
- 时变关系是一种启示。 它们是流处理的体现：流系统被构建出来所希望实现的一切、以及与我们在批处理世界中都知道和喜爱的熟悉工具的强大连接的体现。 从第八章开始会学习到这一点。
- 一个实现良好的分布式流处理引擎是一种魔法。这可以说适用于一般的分布式系统，但是随着您更多地了解这些系统是如何构建以提供它们所做的语义的（从第三章到第五章中的案例），就会更加明显地看到它们为你做了多少繁重的工作。 
- LaTeX/Tikz 是一个制作图表和动画的神奇工具，希望书中在讨论复杂话题时所展示出来的清晰的动画图表可以让更多人去尝试使用 LaTeX/Tikz。

## 线上资源

### 图片

图片在线网址：http://www.streamingbook.net/figures 。

动画图片是通过首先用 LaTeX/Tikz 绘图，渲染成 PDF，再通过 ImageMagick 转成动画 GIF 制作的。绘制动画的完整源代码和说明可以在GitHub上找到：http://github.com/takidau/animations 。但请注意这大约有14000行 LaTeX/Tikz 代码，它们并没有考虑到可能被其他人阅读和使用。

### 代码

尽管书的内容大部分是概念性的，但是仍然使用了一些代码和伪代码片段来帮助阐明要点。代码可以在线获取：http://github.com/takidau/streamingbook 。

由于理解语义是主要目的，代码主要是以 Beam 的 `PTransform/DoFn` 实现提供的，并附带有单元测试。
