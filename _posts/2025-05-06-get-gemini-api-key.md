---
layout: post
title: 申请 Gemini API key 并在 Open WebUI 中配置使用
date: 2025-05-06
categories: 教程
tags: [AI, 自建服务]
media_subpath: /assets/images/get-gemini-api-key/
description: 手把手教你免费申请 Gemini API key，并通过 OpenAI API connections 和 Open WebUI 函数两种方式在 Open WebUI 中配置使用，同时介绍如何开启 Gemini 原生的联网搜索功能。
---

部署好 Open WebUI 之后，下一步就是为它接入语言模型。对于初学者，我最推荐的是 Gemini。

首先，可能也是最重要的一点是，Gemini API 免费，而且只需要一个 Google 账号就能申请，不需要绑卡，也不需要充值，对新手非常友好。

其次，近期发布的 Gemini 2.5 系列模型能力很强——1M token 的上下文窗口（context window），64k token 的输出，内置思考（thinking）能力，还支持[网络搜索](https://ai.google.dev/gemini-api/docs/grounding)（grouding with Google search）。截止 2025 年 4 月 30 日，Gemini 2.5 Pro Preview 在 [Livebench](https://livebench.ai/) 上排名第四，Gemini 2.5 Flash Preview 排名第十。我个人把 Gemini 2.5 Flash Preview 设成 Open WebUI 的默认模型。这是我现在最常用的模型。

![2025 年 4 月 30 日 Livebench 排行榜](1.png){: .shadow w="700" h="381" }
*2025 年 4 月 30 日 Livebench 排行榜*

不过免费也有免费的代价。

首先是[速率限制](https://ai.google.dev/gemini-api/docs/rate-limits#free-tier)（rate limit）。免费用户会受到每分钟请求数（RPM）、每分钟处理 Token 数（TPM）以及每日请求数（RPD）的限制，具体如下图所示：

![免费用户的 Gemini 使用速率限制](2.png){: .shadow w="700" h="523" }
*免费用户的 Gemini 使用速率限制*

对于个人用户来说，这些限制并不算太大的问题，因为一般也用不到这么多。如果你是重度用户，又不想花钱，也可以考虑创建多个 Google 账号并分别申请 API key 轮流使用。反正创建 Google 账号本身不需要花钱。

第二，如果你使用免费版的 Gemini API key，Google 会收集你对话数据用于训练 Gemini 模型。这一点见仁见智，如果对此有所顾虑，可能需要考虑其他选项，比如付费 Gemini API key 或者其它模型提供商的 API。

## 准备工作

在开始之前，你需要先准备：
1. 一个正在运行的 Open WebUI 服务（关于如何部署 Open WebUI，参见[这一篇文章]({% post_url 2025-04-29-deploy-openwebui %})）
2. Open WebUI 服务所在的主机需要配置好代理服务，能够访问到 Gemini
3. 一个 Google 账号。需要注意你的账号是否处于[支持地区](https://ai.google.dev/gemini-api/docs/available-regions)

## 创建 API key

首先，用浏览器访问 [Google AI Studio](https://aistudio.google.com/)。进入页面之后，点击上方的“Get API key”。

![Google AI Studio 主页](3.png){: .shadow w="700" h="456" }
*Google AI Studio 主页*

进入 API keys 页面后，点击“Create API key”，API key 就创建好了。就这么简单，不需要什么复杂操作。

![Google AI Studio 创建 API 页面](4.png){: .shadow w="700" h="456" }
*Google AI Studio 创建 API 页面*

你可以复制 API key，保存到一个地方。当然不复制也行，之后可以在这个页面再次查看你的 API key。

![在 Google AI Studio 创建 API 页面查看 API key](5.png){: .shadow w="700" h="456" }
*在 Google AI Studio 创建 API 页面查看 API key*

> 一定要保管好自己的 API key，不要泄露给任何人。一旦泄露，他人可能滥用你的 API 密钥，导致不必要的麻烦。
{: .prompt-danger }

## 在 Open WebUI 中配置使用 Gemini API key

要在 Open WebUI 中配置使用 Gemini API key，一共有两种方法：
- 第一种是使用 OpenAI API connections
- 第二种是使用 Open WebUI 函数（Function）

如果你只是想快速开始使用 Gemini 模型，我推荐使用第一种方式配置。如果你需要使用 Gemini 原生支持的联网搜索功能，我推荐通过第二种方式配置。

### 第一种方式：使用 OpenAI API connections

由于开发者的设计考量[^1]，Open WebUI 原生只支持 OpenAI 格式的 API 调用，不支持也不打算支持其它提供商的 API key 调用格式（比如 Gemini API key）。但好在 [Gemini API key 原生兼容 OpenAI 格式的 API 调用方式](https://ai.google.dev/gemini-api/docs/openai)，所以我们可以直接通过 OpenAI API Connection 的方式使用它。

在 Open WebUI 的外部连接页面，可以配置 OpenAI API connection。具体配置方法可以参见[这一篇文章的配置 API key 部分]({% post_url 2025-04-29-deploy-openwebui %}#配置-api-key)。

![Open WebUI 外部连接配置页面](6.png){: .shadow w="700" h="473" }
*Open WebUI 外部连接配置页面*

URL 填 `https://generativelanguage.googleapis.com/v1beta/openai`，Key 填你的 API key。Model IDs 部分可以不填，如果不填意味着 Open WebUI 会自动获取所有可用的模型；也可以自己到 [Gemini 模型页面](https://ai.google.dev/gemini-api/docs/models)上去找到自己想用的模型 ID。比如 Gemini 2.5 Flash Preview 04-17 的模型 ID 是 `models/gemini-2.5-flash-preview-04-17`。

配置好之后，点击保存。回到主界面，刷新页面，你就可以看到所有可用的 Gemini 模型了。

### 第二种方式：使用 Open WebUI 函数

通过 OpenAI API connections 配置 Gemini API key 虽然比较简单，容易上手，但这种方式的一个核心缺点是无法调用 Gemini 原生支持的网络搜索功能。根据我的使用体验，Gemini 原生支持的网络搜索功能效果比 Open WebUI 自带的搜索功能（配置 Google PSE）更为出色。所以我更推荐使用 Open WebUI 函数（Function）的方式来配置 Gemini API key。

根据 Open WebUI 官网的介绍，Open WebUI 函数可以理解为 Open WebUI 的内置插件，用以拓展其核心能力[^2]。

你不需要自己写函数，直接使用别人写好的就行。我使用的是 [Gemini Manifold Google_genai](https://openwebui.com/f/suurt8ll/gemini_manifold_google_genai) 函数，它原生支持文生图和网络搜索功能。

#### 0. 检查环境

在创建函数时，Open WebUI 会调用 Open WebUI 所在虚拟环境中的 `pip` 安装函数所需要的依赖。于是，在安装函数之前，你需要保证虚拟环境中有 `pip`。

一般来说虚拟环境中都会有 `pip`，但是使用 `uv` 创建的虚拟环境中没有 `pip`[^3]，你需要手动安装：

```shell
# 在uv创建的虚拟环境下安装pip
uv pip install pip
```

#### 1. 在 Open WebUI 中安装函数

首先来到管理员面板，选择上方的函数，来到函数页面。然后点击右边的加号，添加新函数。

![Open WebUI 函数页面](7.png){: .shadow w="700" h="446" }
*Open WebUI 函数页面*

在创建函数页面，函数名称填 `Gemini Manifold google genai`，函数描述填 `Manifold function for Gemini Developer API. Supports native image generation, grounding with Google Search and streaming. Uses google-genai.`。然后访问 Gemini Manifold Google_genai 函数的[源代码](https://raw.githubusercontent.com/suurt8ll/open_webui_functions/refs/heads/master/plugins/pipes/gemini_manifold.py)，复制所有代码，粘贴到 Open WebUI 页面的函数代码处。

![配置 Gemini Manifold google genai 函数](8.png){: .shadow w="700" h="443" }
*配置 Gemini Manifold google genai 函数，需要填写函数名称、描述和代码*

根据 Gemini Manifold Google_genai 项目的[文档](https://github.com/suurt8ll/open_webui_functions/blob/master/docs/plugins/pipes/gemini_manifold.md)，如果你想使用 Gemini API 自带的网页搜索功能，除了安装 Gemini Manifold Google_genai 函数之外，你还需要安装一个辅助函数 Gemini Manifold Companion。

和刚刚一样，在函数页面创建一个新的函数。函数名填 `Gemini Manifold Companion`，函数描述填 `A companion filter for "Gemini Manifold google_genai" pipe providing enhanced functionality.`。然后访问 Gemini Manifold Companion 的[源代码](https://github.com/suurt8ll/open_webui_functions/raw/refs/heads/master/plugins/filters/gemini_manifold_companion.py)，复制所有代码，粘贴到 Open WebUI 页面的函数代码处。

![配置 Gemini Manifold Companion 辅助函数](9.png){: .shadow w="700" h="586" }
*配置 Gemini Manifold Companion 辅助函数*

（2025-07-28 补充）
配置好之后，你还需要全局应用 Gemini Manifold Companion 辅助函数，让它在全局生效，而不需要在每一个 Gemini 模型页面下手动开启。配置按钮如下图：

![全局应用 Gemini Manifold Companion 辅助函数](12.png){: .shadow w="700" h="228" }
*全局应用 Gemini Manifold Companion 辅助函数*


#### 2. 配置函数的值

创建好一个主函数和一个辅助函数之后，我们还需要配置函数的值（Valves）[^4]，把我们刚刚申请的 Gemini API key 放进去。

配置方式也很简单，来到函数页面，点击 Gemini Manifold google genai 函数右边的齿轮，打开值配置页面。刚打开值配置页面时，所有的值都是“默认”值。点击“默认”就会切换到输入框，可以添加值内容。你只需要关注下面几个值：

- `Gemini Api Key`：填你的 Gemini API key
- `Model Blacklist`：模型黑名单，用于过滤掉你不想看到的模型。我填的是 `gemini-1.0*,gemini-1.5*`，过滤掉 1.0 和 1.5 系列的模型。这两个系列的模型太老了，现在直接用 2.0、2.5 系统的模型就好。
- `Thinking Budget`：用于控制 Gemini 2.5 Flash 的思考长度。这个值越大，模型就思考得越多，但相应的，响应时间也会变得更长。默认值是 `8196`，如果填 `0` 意味着不思考。最高可以填 `24576`
- `Emit Status Updates`：可以看到模型当前的状态（思考中或者是联网搜索中），建议开启

![Gemini Manifold google genai 值配置](10.png){: .shadow w="700" h="690" }
*Gemini Manifold google genai 值配置*

配置好之后，点击保存就大功告成了。

#### 3. 测试

通过这种方式配置 Gemini API key，你可以直接使用 API 自带的联网搜索功能，而不需要额外在 Open WebUI 中配置搜索引擎。你可以返回主页面，试试与模型对话，以及搜索功能。

（2025-07-28 补充）
要使用 Open WebUI 的网络搜索功能，你需要先打开网络搜索功能开关，具体的位置见下图：
![打开 Open WebUI 网络搜索功能](13.png){: .shadow w="700" h="317" }
*打开 Open WebUI 网络搜索功能*

![测试 Gemini 网络搜索功能](11.png){: .shadow w="700" h="447" }
*测试 Gemini 网络搜索功能*

## 总结

总的来说，获取 Gemini API key 非常简单，只需要一个 Google 账号即可。

在 Open WebUI 中配置 Gemini API key 主要有两种方式：
1. 使用 OpenAI API connections：这种方式配置简单快捷，适合想快速上手体验 Gemini 模型的用户。但缺点是无法使用 Gemini 原生的网络搜索等高级功能。
2. 使用 Open WebUI 函数：这种方式配置稍复杂，需要安装 `Gemini Manifold google_genai` 主函数和 `Gemini Manifold Companion` 辅助函数，并进行相应的参数配置。优点是可以充分利用 Gemini API 的原生功能，如网络搜索和文生图，获得更好的使用体验。


## 相关文档

- [Gemini models](https://ai.google.dev/gemini-api/docs/models)：可以查看每个模型的详细信息，包括有哪些模型可以调用，每个模型的上下文窗口大小，输出 token 数量，支持哪些功能，知识截止日期（knowledge cutoff）等等

---

[^1]: 详情请见开发者在 PR 下的评论：[https://github.com/open-webui/open-webui/pull/9241#issuecomment-2629707340](https://github.com/open-webui/open-webui/pull/9241#issuecomment-2629707340)

[^2]: 参见 [https://docs.openwebui.com/features/plugin/functions/](https://docs.openwebui.com/features/plugin/functions/)

[^3]: 可能是因为 `uv` 本身就旨在替代 `pip` 的功能

[^4]: 在 Open WebUI 中，函数的值（Valves）是函数的输入参数。Valves 直译是“阀门”，但不知道为什么官方把它翻译成“值”（Values）。