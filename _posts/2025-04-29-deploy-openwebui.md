---
layout: post
title: 在PVE上使用CT部署Open WebUI服务
date: 2025-04-29
categories: llm
tags: open-webui
---

## 什么是Open WebUI

Open WebUI是一个开源、自托管的Web界面，主要用于与各类大型语言模型（LLM）交互。简单来说，它提供一个类似ChatGPT、Deepseek那样的对话页面，你可以在上面与模型对话。与ChatGPT对话页面不同的是，Open WebUI支持与不同的大模型对话。

一般来说，我们使用某个模型，要去对应提供商的网页去与模型对话。比如说，使用ChatGPT需要访问[chatgpt.com](https://chatgpt.com)；使用Claude，需要访问[claude.ai](https://claude.ai)；使用Deepseek，则要去[chat.deepseek.com](https://chat.deepseek.com)。这种方式不仅需要在不同平台间切换，而且聊天记录分散在各个提供商处，不太方便整理和检索。

Open WebUI的优势在于，只要有对应模型的API，我们就可以直接在Open WebUI上与多个模型对话——不仅仅可以选择某次回答使用哪个模型，我们还可以让多个模型同时回答。而且聊天数据也保存在一个地方，可以随时导入导出。

![Open WebUI界面展示](assets/images/deploy-open-webui-1.png)

## 部署

Open WebUI的[官方文档](https://docs.openwebui.com/getting-started/quick-start/)上提供几种部署方式：Python、Docker、k8s。

我个人比较倾向本机上使用Python安装和部署，而不太喜欢Docker。第一是使用Docker的方式部署，调试不是很方便。其次，使用Docker部署网络配置容易出各种问题，持久存储也需要额外配置。

我现在的部署方案是在PVE中启动一个单独的CT（Container）部署Open WebUI，并通过Cloudflare Tunnel将服务暴露在公网上，再加上Cloudflare Zero Trust进行身份验证。这里先谈谈如何在CT上部署Open WebUI，Cloudflare相关的在后续的blog中再讲。如果你用的不是PVE系统可以跳过PVE相关部分，直接看后面部署的部分。

### 创建CT

要创建一个CT，我们需要先下载一个CT模板（CT template）。在PVE页面上，点进`CT Template`管理页面，具体的路径是：`Datacenter -> <你的节点名> -> local -> CT Templates`，然后点击`Templates`。在`Templates`页面中可以找到官方预设的CT模版。选择一个自己喜欢的模板（我自己选择的是`ubuntu-22.04-standard`），点击右下角的`Download`模板就会开始下载。

下载好CT模板之后，在PVE主页上点击右上角的`Create CT`来创建一个CT。创建之前需要填一些CT相关的参数，我这里记录一些比较重要的参数：
- General页面：
	- Password: 访问CT shell所需要的密码
	- SSH public key（非必需）：如果你想使用ssh，可以把SSH公钥填上去
- Template页面：
	- Template：选择你刚刚下载的CT template
- Disk页面：
	- Disk size（存储）：10GB以上，最好20-30GB
- CPU页面：
	- Cores（CPU核心数）：一般1-2核就足够
- Memory页面：
	- Memory（内存）& Swap：建议2-4GB
- Network页面 & DNS页面：自行配置。如果想使用国外的大模型，需要配置好代理

最后在Comfirm页面确认一遍就可以部署了。

### 下载和运行Open WebUI

创建CT之后，需要先手动启动CT。启动好之后，来到CT页面，通过`Console`登陆CT的shell。这里需要注意用户名填`root`，密码填刚刚你设置的密码。

进到shell，第一件事是更新依赖项：
```shell
apt update && apt upgrade -y
```

然后安装必要的依赖：
```shell
apt install -y curl ffmpeg
```

接下来，我们将使用`uv`来安装Python 和Open WebUI。通过`uv`安装Python 3.11，然后创建虚拟环境，下载Open WebUI:
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

如果你不想装`uv`，也可以通过Python创建虚拟环境，在虚拟环境上安装Open WebUI，效果是一样的。
```shell
python3 -m venv venv
source venv/bin/activate
pip install open-webui
```

最后，通过设置`DATA_DIR`环境变量指定数据存储目录（用于存放数据库），然后启动Open WebUI服务：运行`DATA_DIR=~/.open-webui open-webui serve`。服务启动之后，我们可以在浏览器上通过`http//<主机或者CT的IP地址>:8080`访问它。

### （可选）将Open WebUI部署成一个System Service

为了让Open WebUI能够在系统启动时自动运行，并在意外退出时自动重启，我们可以将其配置为一个`systemd`服务。

以下是在基于Ubuntu/Debian的系统上配置`systemd`服务的步骤。对于其他 Linux 发行版，命令可能略有不同。

在Ubuntu上将服务部署成一个系统服务，我们需要在`/etc/systemd/system`目录下创建一个`.service`文件，告诉`systemd`如何启动、停止和管理这个服务。

运行下面的命令，在`/etc/systemd/system`中创建一个`open-webui.service`文件，并在文件中声明运行所需要的命令、环境变量、重启策略。
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

重载`systemd`的配置、启动Open WebUI服务并且配置开机启动：
```shell
# 重新载配置
systemctl daemon-reload
# 启动服务
systemctl start open-webui.service
# 设置开机启动
systemctl enable open-webui.service
```

启动之后可以查看Open WebUI服务的状态：
```shell
systemctl status open-webui.service
```

正常情况下，你应该会看到服务状态显示为`active (running)`。这说明Open WebUI服务已经正常运行。

## 初步配置

部署好Open WebUI之后，使用浏览器访问`http//<主机或者CT的IP地址>:8080`，来到Open WebUI的主页。首次访问时，系统会要求你创建一个管理员账户。输入用户名、邮箱（格式正确即可，无需真实验证）和密码。

设置好管理员账号，登录，就来到Open WebUI的主页面。

### 中文页面

如果不习惯英文，可以先点右上角的头像，选择"Settings"，点进去。然后在"General"页面把Language改成“简体中文”。

### 配置API key

假设你已经有API Key，你可以来到“管理员面板/Admin panel”（右上角头像 -> 管理员面板/Admin panel），然后点击“设置/Settings“，找到”外部连接/Connection“。

![外部连接（用于配置API key的）页面](assets/images/deploy-open-webui-2.png)

然后在“管理OpenAI API/Manage OpenAI API Connections”连接部分，点击右边的加号，输入相应的API base URL以及API Key，再点击"保存/save"。配置好之后返回主页面，就能看到相应的模型了。

## 运维

部署好服务之后，后续还需要运维。这里记录一下常见的运维命令：
- 查看服务状态：`systemctl status open-webui.service`
- 重启服务：`systemctl restart open-webui.service`
- 停止服务：`systemctl stop open-webui.service`
- 查看Open WebUI的日志：`journalctl -u open-webui.service -f`
- 升级Open WebUI到最新版本并重启服务：`uv pip install open-webui --upgrade && systemctl restart open-webui.service`

## 总结

至此，我们已经在PVE的CT上完成了Open WebUI的安装与运行，并演示了如何将其配置为`systemd`服务，实现开机自启与故障自动重启。最后还记录了一些常见的运维命令。