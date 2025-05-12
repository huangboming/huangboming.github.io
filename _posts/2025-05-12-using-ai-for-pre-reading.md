---
layout: post
title: 我的AI略读方法论
date: 2025-05-12
categories: llm
tags: open-webui reading prompts
---

我习惯用RSS订阅各种信息源，并且每天花一些时间筛选我感兴趣的内容来阅读。为此，我还为自己制作了一份[Hugging face daily paper的RSS feed](https://github.com/huangboming/huggingface-daily-paper-feed)。

RSS的主要问题是信息过载——每天动辄几十上百条新内容推送。我的对策是：
1. 不为推送信息过多而感到焦虑。一次读不完所有推送也无妨，可以之后再读。
2. 每天花一段固定的时间，从中挑选出我感兴趣的内容。

这样筛选一轮下来，每天仍有5到20篇新闻、博客或论文等着我。这个数量其实也不少，要把它们通读一遍相当花时间。于是，我想了一个办法：让AI帮助我略读——总结文章的核心内容、思想。略读之后，我再决定要不要精读或者记录文献笔记。在精读时，碰到什么不理解的地方还可以随时请教AI。

## 略读提示词

下面是我设计的AI略读提示词：

```markdown
Write a comprehensive and structured summary (1,000-2,000 words) of the provided text, ensuring clarity, depth, and coherence. The summary should be structured around the following four analytical dimensions:

1. Structural Overview: Clearly outline the main theme, central argument, and overall structure of the text.
2. Analytical Breakdown: Explain how the author supports their arguments, including key concepts, evidence, reasoning, and examples.
3. Critical Evaluation: Assess the text’s persuasiveness, strengths, weaknesses, biases, and alternative perspectives. Maintain an objective tone unless otherwise specified.
4. Practical Application: Discuss how the text’s ideas can be applied in real life, considering relevance, impact, and actionable insights.

Adapt the summary style to the nature of the text (e.g., academic tone for research papers, engaging style for blogs). For academic works, cite key arguments with paraphrased points or direct quotes where appropriate. If summarizing a narrative or literary work, maintain fidelity to its themes and tone.

## Constraints:

- Maintain an objective, third-person tone unless explicitly instructed otherwise.
- Avoid excessive subjective interpretation unless required.
- Ensure coherence between sections to create a seamless, logically flowing summary.
```

这个提示词的设计借鉴了《如何阅读一本书》中的思想——要了解一本书，你需要回答四个主要问题[^1]：
1. 整体来看，这本书到底在谈些什么？找到这本书的主题、核心论点、以及整本书的论述架构
2. 具体来看，作者怎么论述书中的论点？
3. 从批判性的角度来看，这本书说得有道理吗？
4. 从应用的角度来看，这本书对我有什么用？怎么应用到现实中？

我更推荐使用英文提示词，主要有两个原因：一是AI模型通常接受了更多的英文训练，因此能更准确地理解英文指令中的细节。二是英文提示词所消耗的token数量更少，成本更低。

把这份提示词以及你要阅读的文本一并发送给AI，它会生成一份1000到2000词的英文总结。我们并不需要阅读这份总结[^2]——它的真正目的是引导AI深度理解原文，并提炼核心内容。

最关键的一步是，AI生成总结后，追加一句“用大白话解释“[^3]，AI就会用通俗易懂的大白话为你解释文章的主要内容。

“用大白话解释“是一个颇具魔力的提示词。它能让AI使用简单明了的语言解释复杂概念，就像一位朋友在和你聊天一样，从而极大地降低阅读理解的难度。它可以用在很多地方，比如你让AI给你解释一个复杂概念时，如果你觉得它还没解释清楚，不妨试试“用大白话解释”。

## 示例一：在Open WebUI中阅读网页

阅读博客、新闻这类网页时，推荐使用Open WebUI自带的网页抓取功能，先抓取网页中的文本[^4]，再交AI总结。

操作很简单：在Open WebUI对话框中输入`#`，紧接着粘贴或输入目标网址，回车即可。Open WebUI便会自动抓取页面文本。比如说，要阅读[这篇博客](https://notashelf.dev/posts/curse-of-knowing)，可以在对话框中输入`#https://notashelf.dev/posts/curse-of-knowing`，然后回车。

![输入# + 文章URL](assets/images/using-ai-for-pre-reading-1.png)
*输入“#”和目标网址*

![输入# + 文章URL后回车](assets/images/using-ai-for-pre-reading-2.png)
*回车后，Open WebUI自动抓取网页内容*

接着，将前述的AI略读提示词输入对话框并发送，让AI先理解、总结文章：

![输入AI略读提示词](assets/images/using-ai-for-pre-reading-3.png)
*发送AI略读提示词*


最后，再补上一句“用大白话解释”：

![输入“用大白话解释”](assets/images/using-ai-for-pre-reading-4.png)
*发送“用大白话解释”指令后，AI输出通俗易懂的解释*

可以看到AI生成的解释通俗易懂，核心内容一目了然。

## 示例二：在Google AI studio中阅读pdf

对于PDF格式的内容（比如学术论文），如果你没有配置好Open WebUI的Content Extraction Engine以及RAG功能，我更推荐直接在[Google AI studio](https://aistudio.google.com)上阅读。它会通过OCR技术自动提取PDF文本[^5]，不需要你自己提取，非常方便。

步骤也同样简单：访问AI Studio，上传PDF，然后依次使用AI略读提示词和“用大白话解释”。

下面以[The Leaderboard Illusion](https://arxiv.org/abs/2504.20879)这篇论文为例。

![上传pdf文件](assets/images/using-ai-for-pre-reading-5.png)
*在Google AI Studio上传PDF文件*

![AI略读提示词 + “用大白话解释”之后的效果](assets/images/using-ai-for-pre-reading-6.png)
*使用AI略读提示词及“用大白话解释”指令后的输出效果*

## 总结

以上就是我运用AI进行略读的方法：先用精心设计的提示词引导AI深度理解文本，再用一句“用大白话解释”获取通俗易懂的核心内容。

---

[^1]: 莫提默 J. 艾得勒, 查尔斯 范多伦著 ; 郝明义, 朱衣译. 如何阅读一本书. 第一版. 北京: 商务印书馆, 2004. 这四个主要问题可以在第五章找到

[^2]: 这份总结很可能比原文长，读它不如直接去读原文

[^3]: 我从和菜头的一篇文章中发现了这个神奇的提示词，详见：[deepseek进阶使用指南](https://www.hecaitou.com/2025/01/Deepseek-Advanced-Usage-Guide.html)

[^4]: 只抓取文本内容，并不会抓取图片。对于多图且图片对于内容很重要的网页，建议导出成pdf，在AI studio中阅读

[^5]: AI studio文档中没有明确说明它用OCR提取pdf中的文本内容。我在一次实验中发现这一点：上传一个pdf文档，然后输入“output the first page of the pdf, do not change anything I provided to you“。Gemini输出的开头结尾会带有“Start of OCR for page 1“和“End of OCR for page 1“

