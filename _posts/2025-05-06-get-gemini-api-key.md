---
layout: post
title: 申请Gemini API key并在Open WebUI中配置使用
date: 2025-05-06
categories: llm
tags: open-webui
---

部署好Open WebUI之后，下一步就是为它接入语言模型。对于初学者，我最推荐的是Gemini。

首先，可能也是最重要的一点是，Gemini API免费，而且只需要一个Google账号就能申请，不需要绑卡，也不需要充值，对新手非常友好。

其次，近期发布的Gemini 2.5系列模型能力很强——1M token的上下文窗口（context window），64k token的输出，内置思考（thinking）能力，还支持[网络搜索](https://ai.google.dev/gemini-api/docs/grounding)（grouding with Google search）。截止2025年4月30日，Gemini 2.5 Pro Preview在[Livebench](https://livebench.ai/)上排名第四，Gemini 2.5 Flash Preview排名第十。我个人把Gemini 2.5 Flash Preview设成Open WebUI的默认模型。这是我现在最常用的模型。

![2025年4月30日Livebench排行榜](assets/images/get-gemini-api-key/1.png)
*2025年4月30日Livebench排行榜*

不过免费也有免费的代价。

首先是[速率限制](https://ai.google.dev/gemini-api/docs/rate-limits#free-tier)（rate limit）。免费用户会受到每分钟请求数（RPM）、每分钟处理Token数（TPM）以及每日请求数（RPD）的限制，具体如下图所示：

![免费用户的Gemini使用速率限制](assets/images/get-gemini-api-key/2.png)
*免费用户的Gemini使用速率限制*

对于个人用户来说，这些限制并不算太大的问题，因为一般也用不到这么多。如果你是重度用户，又不想花钱，也可以考虑创建多个Google账号并分别申请API key轮流使用。反正创建Google账号本身不需要花钱。

第二，如果你使用免费版的Gemini API key，Google会收集你对话数据用于训练Gemini模型。这一点见仁见智，如果对此有所顾虑，可能需要考虑其他选项，比如付费Gemini API key或者其它模型提供商的API。

## 准备工作

在开始之前，你需要先准备：
1. 一个正在运行的Open WebUI服务（关于如何部署Open WebUI，参见[这一篇文章]({% post_url 2025-04-29-deploy-openwebui %})）
2. Open WebUI服务所在的主机需要配置好代理服务，能够访问到Gemini
3. 一个Google账号。需要注意你的账号是否处于[支持地区](https://ai.google.dev/gemini-api/docs/available-regions)

## 创建API key

首先，用浏览器访问[Google AI Studio](https://aistudio.google.com/)。进入页面之后，点击上方的"Get API key"。

![Google AI Studio主页](assets/images/get-gemini-api-key/3.png)
*Google AI Studio主页*

进入API keys页面后，点击"Create API key"，API key就创建好了。就这么简单，不需要什么复杂操作。

![Google AI Studio创建API页面](assets/images/get-gemini-api-key/4.png)
*Google AI Studio创建API页面*

你可以复制API key，保存到一个地方。当然不复制也行，之后可以在这个页面再次查看你的API key。

![在Google AI Studio创建API页面查看API key](assets/images/get-gemini-api-key/5.png)
*在Google AI Studio创建API页面查看API key*

需要注意，一定要保管好自己的API，不要泄露给任何人。一旦泄露，他人可能滥用你的API密钥，导致不必要的麻烦。

## 在Open WebUI中配置使用Gemini API key

要在Open WebUI中配置使用Gemini API key，一共有两种方法：
- 第一种是使用OpenAI API connections
- 第二种是使用Open WebUI函数（Function）

如果你只是想快速开始使用Gemini模型，我推荐使用第一种方式配置。如果你需要使用Gemini原生支持的联网搜索功能，我推荐通过第二种方式配置。

### 第一种方式：使用OpenAI API connections

由于开发者的设计考量[^1]，Open WebUI原生只支持OpenAI格式的API调用，不支持也不打算支持其它提供商的API key调用格式（比如Gemini API key）。但好在[Gemini API key原生兼容OpenAI格式的API调用方式](https://ai.google.dev/gemini-api/docs/openai)，所以我们可以直接通过OpenAI API Connection的方式使用它。

在Open WebUI的外部连接页面，可以配置OpenAI API connection。具体配置方法可以参见[这一篇文章的配置API key部分]({% post_url 2025-04-29-deploy-openwebui %}#配置api-key)。

![Open WebUI外部连接配置页面](assets/images/get-gemini-api-key/6.png)
*Open WebUI外部连接配置页面*

URL填`https://generativelanguage.googleapis.com/v1beta/openai`，Key填你的API key。Model IDs部分可以不填，如果不填意味着Open WebUI会自动获取所有可用的模型；也可以自己到[Gemini模型页面](https://ai.google.dev/gemini-api/docs/models)上去找到自己想用的模型ID。比如Gemini 2.5 Flash Preview 04-17的模型ID是`models/gemini-2.5-flash-preview-04-17`。

配置好之后，点击保存。回到主界面，刷新页面，你就可以看到所有可用的Gemini模型了。

### 第二种方式：使用Open WebUI函数

通过OpenAI API connections配置Gemini API key虽然比较简单，容易上手，但这种方式的一个核心缺点是无法调用Gemini原生支持的网络搜索功能。根据我的使用体验，Gemini原生支持的网络搜索功能效果比Open WebUI自带的搜索功能（配置Google PSE）更为出色。所以我更推荐使用Open WebUI函数（Function）的方式来配置Gemini API key。

根据Open WebUI官网的介绍，Open WebUI函数可以理解为Open WebUI的内置插件，用以拓展其核心能力[^2]。

你不需要自己写函数，直接使用别人写好的就行。我使用的是[Gemini Manifold Google_genai](https://openwebui.com/f/suurt8ll/gemini_manifold_google_genai)函数，它原生支持文生图和网络搜索功能。

#### 0. 检查环境

在创建函数时，Open WebUI会调用Open WebUI所在虚拟环境中的`pip`安装函数所需要的依赖。于是，在安装函数之前，你需要保证虚拟环境中有`pip`。

一般来说虚拟环境中都会有`pip`，但是使用`uv`创建的虚拟环境中没有`pip`[^3]，你需要手动安装：

```shell
# 在uv创建的虚拟环境下安装pip
uv pip install pip
```

#### 1. 在Open WebUI中安装函数

首先来到管理员面板，选择上方的函数，来到函数页面。然后点击右边的加号，添加新函数。

![Open WebUI函数页面](assets/images/get-gemini-api-key/7.png)
*Open WebUI函数页面*

在创建函数页面，函数名称填`Gemini Manifold google genai`，函数描述填`Manifold function for Gemini Developer API. Supports native image generation, grounding with Google Search and streaming. Uses google-genai.`。然后访问Gemini Manifold Google_genai函数的[源代码](https://raw.githubusercontent.com/suurt8ll/open_webui_functions/refs/heads/master/plugins/pipes/gemini_manifold.py)，复制所有代码，粘贴到Open WebUI页面的函数代码处。

![配置Gemini Manifold google genai函数](assets/images/get-gemini-api-key/8.png)
*配置Gemini Manifold google genai函数，需要填写函数名称、描述和代码*

根据Gemini Manifold Google_genai项目的[文档](https://github.com/suurt8ll/open_webui_functions/blob/master/docs/plugins/pipes/gemini_manifold.md)，如果你想使用Gemini API自带的网页搜索功能，除了安装Gemini Manifold Google_genai函数之外，你还需要安装一个辅助函数Gemini Manifold Companion。

和刚刚一样，在函数页面创建一个新的函数。函数名填`Gemini Manifold Companion`，函数描述填`A companion filter for "Gemini Manifold google_genai" pipe providing enhanced functionality.`。然后访问Gemini Manifold Companion的[源代码](https://github.com/suurt8ll/open_webui_functions/raw/refs/heads/master/plugins/filters/gemini_manifold_companion.py)，复制所有代码，粘贴到Open WebUI页面的函数代码处。

![配置Gemini Manifold Companion辅助函数](assets/images/get-gemini-api-key/9.png)
*配置Gemini Manifold Companion辅助函数*

（2025-07-28补充）
配置好之后，你还需要全局应用Gemini Manifold Companion辅助函数，让它在全局生效，而不需要在每一个Gemini模型页面下手动开启。配置按钮如下图：

![全局应用Gemini Manifold Companion辅助函数](assets/images/get-gemini-api-key/12.png)
*全局应用Gemini Manifold Companion辅助函数*


#### 2. 配置函数的值

创建好一个主函数和一个辅助函数之后，我们还需要配置函数的值（Valves）[^4]，把我们刚刚申请的Gemini API key放进去。

配置方式也很简单，来到函数页面，点击Gemini Manifold google genai函数右边的齿轮，打开值配置页面。刚打开值配置页面时，所有的值都是"默认"值。点击"默认"就会切换到输入框，可以添加值内容。你只需要关注下面几个值：

- `Gemini Api Key`：填你的Gemini API key
- `Model Blacklist`：模型黑名单，用于过滤掉你不想看到的模型。我填的是`gemini-1.0*,gemini-1.5*`，过滤掉1.0和1.5系列的模型。这两个系列的模型太老了，现在直接用2.0、2.5系统的模型就好。
- `Thinking Budget`：用于控制Gemini 2.5 Flash的思考长度。这个值越大，模型就思考得越多，但相应的，响应时间也会变得更长。默认值是`8196`，如果填`0`意味着不思考。最高可以填`24576`
- `Emit Status Updates`：可以看到模型当前的状态（思考中或者是联网搜索中），建议开启

![Gemini Manifold google genai值配置](assets/images/get-gemini-api-key/10.png)
*Gemini Manifold google genai值配置*

配置好之后，点击保存就大功告成了。

#### 3. 测试

通过这种方式配置Gemini API key，你可以直接使用API自带的联网搜索功能，而不需要额外在Open WebUI中配置搜索引擎。你可以返回主页面，试试与模型对话，以及搜索功能。

（2025-07-28补充）
要使用Open WebUI的网络搜索功能，你需要先打开网络搜索功能开关，具体的位置见下图：
![打开Open WebUI网络搜索功能](assets/images/get-gemini-api-key/13.png)
*打开Open WebUI网络搜索功能*

![测试Gemini网络搜索功能](assets/images/get-gemini-api-key/11.png)
*测试Gemini网络搜索功能*

## 总结

总的来说，获取Gemini API key非常简单，只需要一个Google账号即可。

在Open WebUI中配置Gemini API key主要有两种方式：
1. 使用OpenAI API connections：这种方式配置简单快捷，适合想快速上手体验Gemini模型的用户。但缺点是无法使用Gemini原生的网络搜索等高级功能。
2. 使用Open WebUI函数：这种方式配置稍复杂，需要安装`Gemini Manifold google_genai`主函数和`Gemini Manifold Companion`辅助函数，并进行相应的参数配置。优点是可以充分利用Gemini API的原生功能，如网络搜索和文生图，获得更好的使用体验。


## 相关文档

- [Gemini models](https://ai.google.dev/gemini-api/docs/models)：可以查看每个模型的详细信息，包括有哪些模型可以调用，每个模型的上下文窗口大小，输出token数量，支持哪些功能，知识截止日期（knowledge cutoff）等等

---

[^1]: 详情请见开发者在pr下的评论：[https://github.com/open-webui/open-webui/pull/9241#issuecomment-2629707340](https://github.com/open-webui/open-webui/pull/9241#issuecomment-2629707340)

[^2]: 参见[https://docs.openwebui.com/features/plugin/functions/](https://docs.openwebui.com/features/plugin/functions/)

[^3]: 可能是因为`uv`本身就旨在替代`pip`的功能

[^4]: 在Open WebUI中，函数的值（Valves）是函数的输入参数。Valves直译是"阀门"，但不知道为什么官方把它翻译成"值"（Values）。