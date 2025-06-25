---
layout: post
title: 在 Open WebUI 中使用 Gemini 2.5 TTS 模型
date: 2025-06-25
categories: llm
tags: open-webui
---

在今年的 I/O 大会上，Google 推出了 Gemini 2.5 系列 TTS (text-to-speech) 模型的预览版，包括 Gemini 2.5 Flash 和 Gemini 2.5 Pro。用户可以在 [AI Studio](https://aistudio.google.com/generate-speech) 上免费体验。免费层级的 AI Studio API 用户也有一定的试用额度，具体可参见[官方文档](https://ai.google.dev/gemini-api/docs/rate-limits#free-tier)。

初步体验过后，我发现它生成的语音流畅、自然，几乎听不出明显的机器音。虽然官方文档上没有明确说明支持中文，但实测下来发现可用，而且效果还可以。

我最近一直在使用 Open WebUI，它内置的 TTS 功能可以朗读对话内容。于是我萌生了将 Gemini 2.5 TTS 集成进 Open WebUI 的想法。经过一番研究，我发现 Open WebUI 并不支持 Gemini 的 API 格式，但兼容 OpenAI 的格式。

于是，我借助 AI 快速开发了一个简单的[转换服务](https://github.com/huangboming/gemini-to-openai-tts)，它能将 OpenAI 格式的 TTS 请求转换为 Gemini API 所需的格式，然后调用 Gemini API 获取音频并返回。通过这个服务，我们就能在 Open WebUI 中无缝使用 Gemini 2.5 TTS 模型。如果你觉得这个项目对你有帮助，欢迎在 GitHub 上给我一个 star。

## 部署服务

### 使用 Docker

首先，你需要创建一个 `.env` 文件来配置环境变量（可参考项目中的 [.env.example](https://github.com/huangboming/gemini-to-openai-tts/blob/main/.env.example) 文件）：
```
# .env.example - 环境变量配置模板
# 复制此文件为 .env 并填入你的实际配置

# === 服务认证配置 ===
# 客户端访问此服务时需要提供的 API 密钥（可以是多个，用逗号分隔）
# 建议使用强密码生成器生成 32+ 位的随机字符串
API_KEYS="your_secret_api_key_1,your_secret_api_key_2"

# === 上游 API 配置 ===
# 你的 Google Gemini API 密钥
# 获取方式：https://aistudio.google.com/apikey
GEMINI_API_KEY="your_google_gemini_api_key_here"

# === 应用配置 ===
# 日志级别：DEBUG, INFO, WARNING, ERROR
# 开发环境推荐使用 DEBUG，生产环境推荐使用 INFO
LOG_LEVEL=INFO
```


配置完成后，可以直接使用我打包好的镜像，使用以下命令即可一键运行服务：
```shell
docker run -d \
  --name gemini-tts \
  -p 8000:8000 \
  --env-file .env \
  --restart unless-stopped \
  boming/gemini-to-openai-tts:latest
```

### 本机部署

如果你希望在本地直接运行，步骤同样很简单。项目提供了一个 Makefile 来简化流程：
1. 克隆仓库： `git clone https://github.com/huangboming/gemini-to-openai-tts`
2. 进入项目目录： `cd gemini-to-openai-tts`
3. 配置环境变量： `cp .env.example .env`，然后修改 `.env` 文件，具体说明同上一节。
4. 安装依赖：`make install`
5. 运行服务： `make run`

服务启动后，你可以在 `http://<你的IP>:8000` 访问它。

## 配置 Open WebUI

Open WebUI 的配置过程与设置标准的 OpenAI TTS 类似，只需修改 API 端点和密钥即可。

具体配置如下：
1. 在 Open WebUI 中打开管理员面板
2. 导航至 `设置 -> 语音`
3. 配置以下设置：
	- 将 TTS API 端点设置为你的服务 URL（例如：`http://localhost:8000/v1/audio/speech`）
	- 在认证字段中添加你的 API key，这里是上面环境变量中 `API_KEYS` 中的一个
	- 从可用选项中选择你喜欢的语音

一个配置示例如下：

![Open WebUI TTS 配置示例](assets/images/using-gemini-tts-in-open-webui/1.png)

最后有一个小提示：如果你使用的是免费层级的 Gemini API key，建议将“拆分回复“选项设置为 `paragraphs` (段落) 或 `none` (不拆分)。因为免费版 API 每分钟的请求次数有限（3次/分钟，15次/天），如果按句子拆分，很容易超出额度。