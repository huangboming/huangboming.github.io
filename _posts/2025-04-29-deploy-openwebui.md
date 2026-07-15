---
layout: post
title: 在 PVE 上使用 CT 部署 Open WebUI 服务
date: 2025-04-29
categories: 教程
tags: [AI, 自建服务]
description: 在 Proxmox VE 的 CT 容器中部署 Open WebUI，并配置为 systemd 服务实现开机自启与故障自动重启。
media_subpath: /assets/images/deploy-openwebui/
---

## 什么是 Open WebUI

Open WebUI 是一个开源、自托管的 Web 界面，主要用于与各类大型语言模型（LLM）交互。简单来说，它提供一个类似 ChatGPT、Deepseek 那样的对话页面，你可以在上面与模型对话。与 ChatGPT 对话页面不同的是，Open WebUI 支持与不同的大模型对话。

一般来说，我们使用某个模型，要去对应提供商的网页去与模型对话。比如说，使用 ChatGPT 需要访问 [chatgpt.com](https://chatgpt.com)；使用 Claude，需要访问 [claude.ai](https://claude.ai)；使用 Deepseek，则要去 [chat.deepseek.com](https://chat.deepseek.com)。这种方式不仅需要在不同平台间切换，而且聊天记录分散在各个提供商处，不太方便整理和检索。

Open WebUI 的优势在于，只要有对应模型的 API，我们就可以直接在 Open WebUI 上与多个模型对话——不仅仅可以选择某次回答使用哪个模型，我们还可以让多个模型同时回答。而且聊天数据也保存在一个地方，可以随时导入导出。

![Open WebUI 界面展示](1.png){: .shadow w="700" h="434" }
*Open WebUI 主界面*

## 部署

Open WebUI 的[官方文档](https://docs.openwebui.com/getting-started/quick-start/)上提供几种部署方式：Python、Docker、k8s。

我个人比较倾向本机上使用 Python 安装和部署，而不太喜欢 Docker。第一是使用 Docker 的方式部署，调试不是很方便。其次，使用 Docker 部署网络配置容易出各种问题，持久存储也需要额外配置。

我现在的部署方案是在 PVE 中启动一个单独的 CT（Container）部署 Open WebUI，并通过 Cloudflare Tunnel 将服务暴露在公网上，再加上 Cloudflare Zero Trust 进行身份验证。这里先谈谈如何在 CT 上部署 Open WebUI，Cloudflare 相关的在后续的 blog 中再讲。如果你用的不是 PVE 系统可以跳过 PVE 相关部分，直接看后面部署的部分。

### 创建 CT

要创建一个 CT，我们需要先下载一个 CT 模板（CT template）。在 PVE 页面上，点进 `CT Template` 管理页面，具体的路径是：`Datacenter -> <你的节点名> -> local -> CT Templates`，然后点击 `Templates`。在 `Templates` 页面中可以找到官方预设的 CT 模版。选择一个自己喜欢的模板（我自己选择的是 `ubuntu-22.04-standard`），点击右下角的 `Download` 模板就会开始下载。

下载好 CT 模板之后，在 PVE 主页上点击右上角的 `Create CT` 来创建一个 CT。创建之前需要填一些 CT 相关的参数，我这里记录一些比较重要的参数：
- General 页面：
	- Password: 访问 CT shell 所需要的密码
	- SSH public key（非必需）：如果你想使用 ssh，可以把 SSH 公钥填上去
- Template 页面：
	- Template：选择你刚刚下载的 CT template
- Disk 页面：
	- Disk size（存储）：10GB 以上，最好 20-30GB
- CPU 页面：
	- Cores（CPU 核心数）：一般 1-2 核就足够
- Memory 页面：
	- Memory（内存）& Swap：建议 2-4GB
- Network 页面 & DNS 页面：自行配置。如果想使用国外的大模型，需要配置好代理

最后在 Comfirm 页面确认一遍就可以部署了。

### 下载和运行 Open WebUI

创建 CT 之后，需要先手动启动 CT。启动好之后，来到 CT 页面，通过 `Console` 登陆 CT 的 shell。这里需要注意用户名填 `root`，密码填刚刚你设置的密码。

进到 shell，第一件事是更新依赖项：
```shell
apt update && apt upgrade -y
```

然后安装必要的依赖：
```shell
apt install -y curl ffmpeg
```

接下来，我们将使用 `uv` 来安装 Python 和 Open WebUI。通过 `uv` 安装 Python 3.11，然后创建虚拟环境，下载 Open WebUI:
```shell
# 安装uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 安装Python 3.11
uv python install 3.11
# 创建虚拟环境
uv venv .venv --python 3.11
source .venv/bin/activate
# 安装open webui
uv pip install open-webui
```

如果你不想装 `uv`，也可以通过 Python 创建虚拟环境，在虚拟环境上安装 Open WebUI，效果是一样的。
```shell
python3 -m venv venv
source venv/bin/activate
pip install open-webui
```

最后，通过设置 `DATA_DIR` 环境变量指定数据存储目录（用于存放数据库），然后启动 Open WebUI 服务：运行 `DATA_DIR=~/.open-webui open-webui serve`。服务启动之后，我们可以在浏览器上通过 `http://<主机或者CT的IP地址>:8080` 访问它。

### （可选）将 Open WebUI 部署成一个 System Service

为了让 Open WebUI 能够在系统启动时自动运行，并在意外退出时自动重启，我们可以将其配置为一个 `systemd` 服务。

以下是在基于 Ubuntu/Debian 的系统上配置 `systemd` 服务的步骤。对于其他 Linux 发行版，命令可能略有不同。

在 Ubuntu 上将服务部署成一个系统服务，我们需要在 `/etc/systemd/system` 目录下创建一个 `.service` 文件，告诉 `systemd` 如何启动、停止和管理这个服务。

运行下面的命令，在 `/etc/systemd/system` 中创建一个 `open-webui.service` 文件，并在文件中声明运行所需要的命令、环境变量、重启策略。
```shell
cat <<EOF > /etc/systemd/system/open-webui.service
[Unit]
Description=Open WebUI Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root
Environment="DATA_DIR=/root/.open-webui"
ExecStart=/bin/bash -l -c "source /root/.venv/bin/activate && open-webui serve"
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

重载 `systemd` 的配置、启动 Open WebUI 服务并且配置开机启动：
```shell
# 重新载配置
systemctl daemon-reload
# 启动服务
systemctl start open-webui.service
# 设置开机启动
systemctl enable open-webui.service
```

启动之后可以查看 Open WebUI 服务的状态：
```shell
systemctl status open-webui.service
```

正常情况下，你应该会看到服务状态显示为 `active (running)`。这说明 Open WebUI 服务已经正常运行。

## 初步配置

部署好 Open WebUI 之后，使用浏览器访问 `http://<主机或者CT的IP地址>:8080`，来到 Open WebUI 的主页。首次访问时，系统会要求你创建一个管理员账户。输入用户名、邮箱（格式正确即可，无需真实验证）和密码。

设置好管理员账号，登录，就来到 Open WebUI 的主页面。

### 中文页面

如果不习惯英文，可以先点右上角的头像，选择“Settings”，点进去。然后在“General”页面把 Language 改成“简体中文”。

### 配置 API key

假设你已经有 API Key，你可以来到“管理员面板/Admin panel”（右上角头像 -> 管理员面板/Admin panel），然后点击“设置/Settings”，找到“外部连接/Connection”。

![外部连接页面](2.png){: .shadow w="700" h="423" }
*进入 Open WebUI 外部连接页面*

然后在“管理 OpenAI API/Manage OpenAI API Connections”连接部分，点击右边的加号，输入相应的 API base URL 以及 API Key，再点击“保存/save”。配置好之后返回主页面，就能看到相应的模型了。

## 运维

部署好服务之后，后续还需要运维。这里记录一下常见的运维命令：
- 查看服务状态：`systemctl status open-webui.service`
- 重启服务：`systemctl restart open-webui.service`
- 停止服务：`systemctl stop open-webui.service`
- 查看 Open WebUI 的日志：`journalctl -u open-webui.service -f`
- 升级 Open WebUI 到最新版本并重启服务：`uv pip install open-webui --upgrade && systemctl restart open-webui.service`
