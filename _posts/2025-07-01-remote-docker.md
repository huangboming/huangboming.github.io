---
layout: post
title: 使用 Docker Context + SSH 访问远端 Docker Daemon
date: 2025-07-01
categories: 开发环境
tags: docker 开发环境 DevOps
---

在 Windows 或者 macOS 上使用 Docker Desktop 时，Docker Desktop 会创建一个轻量级虚拟机来运行 Docker 守护进程（Docker daemon， `dockerd`）。这相当消耗内存和 CPU 资源。我在 Mac 上只是启动 Docker Desktop，没有运行任何 container 就消耗了 2GB 内存。

而且，我在 M 系列芯片的 Mac 上使用 Docker 时，经常碰到镜像架构不兼容问题。打包出的镜像是 ARM 架构，无法直接部署到 x86 服务器上。虽然可以通过 [Docker Buildx](https://github.com/docker/buildx) 插件跨架构编译，但跨架构编译通常比原生编译更慢，配置也更繁琐。

恰好我家里有一台 x86 的 PVE 主机，其中跑着一台 Ubuntu 虚拟机。我就想，能不能让 Mac 上的 `docker` 客户端直接操作 Ubuntu 虚拟机的 `dockerd`，在上面构建镜像、部署服务。

答案是肯定的，而且相当优雅 —— Docker Context + SSH。

## Docker 的 C/S 架构

要理解这个方案，我们得先回到 Docker 的架构：它是一个客户端/服务器（client-server, c/s）架构。

我们日常在终端里使用的 `docker` 命令，其实是 Docker 客户端。它负责将指令发送给 Docker 守护进程（`dockerd`），后者则负责管理镜像、容器、网络和存储等资源。

![docker 架构图](https://docs.docker.com/get-started/images/docker-architecture.webp)

既然它是 c/s 架构，那么客户端与守护进程就不必非得在同一台机器上。我们完全可以让本地的 `docker` 客户端，通过网络，与远程服务器上的 `dockerd` 通信。

传统方式是暴露 `dockerd` 的 TCP 端口并用 TLS 证书加密。具体配置方式可以参考[这篇博文](https://blog.itshangxp.com/docker/docker-remote/)。这套配置相对繁琐，需要在服务器端修改 `dockerd` 配置，还要生成和管理证书文件。

相比之下，SSH 方案要便捷得多。我们只需要在本机配置好 SSH 密钥，无需对远程服务器上的 Docker 做任何改动，既安全又方便。

## 1. 配置 SSH 密钥

Docker 通过 SSH 连接远程主机时，只支持密钥认证，不支持密码认证。所以，在开始之前，你先需要配置 SSH 密钥登录远端服务器。

下面以一台 IP 地址为 `192.168.31.100`，用户名为 `ubuntu` 的远程 Ubuntu 虚拟机为例。

首先，我们要在本地主机生成一对公私钥。公钥会被安装到远端服务器上，私钥则用于身份认证。这里推荐使用更现代、更安全的 `ed25519` 算法：

```shell
# ssh-keygen -t <算法> -f <公私钥路径>
ssh-keygen -t ed25519 -f ~/.ssh/ubuntu
```

生成过程中，系统会提示你设置一个密码短语（pass phrase）。这是一个为私钥本身加密的密码。设置后，每次使用该私钥时，都需要先输入pass phrase。如果嫌麻烦也可以不设置。当然，代价是牺牲了一定的安全性，一旦私钥文件泄露，攻击者就能直接访问你的服务器。

完成后，你的 `~/.ssh/` 目录下会新增两个文件：
- `ubuntu`：私钥文件
- `ubuntu.pub`：公钥文件

然后，我们要将公钥添加到远端服务器：

```shell
# ssh-copy-id -i <公钥路径> <服务器用户名>@<服务器主机地址>
ssh-copy-id -i ~/.ssh/ubuntu.pub ubuntu@192.168.31.100
```

这条命令会自动将你的公钥内容追加到服务器的 `~/.ssh/authorized_keys` 文件中。期间会要求你输入一次服务器的登录密码，授权本次公钥写入操作。

完成后，你可以测试一下是否能用私钥直接登录：（如果设置了 pass phrase，则需要输入它）：

```shell
# ssh -i <私钥路径> <服务器用户名>@<服务器主机地址>。
ssh -i ~/.ssh/ubuntu ubuntu@192.168.31.100
```

如果能成功登录，说明密钥已配置成功。

每次都输入长长的命令（`ssh -i ...`）显然不够优雅，我们可以通过配置 `~/.ssh/config` 文件，为服务器创建一个别名，简化登录命令。

编辑（如果不存在则创建）`~/.ssh/config` 文件，在文件末尾添加以下内容：

```
# Host <服务器别名>
#     HostName <服务器地址>
#     User <服务器用户名>
#     IdentityFile <私钥文件地址>

# 比如
Host ubuntu
    HostName 192.168.31.100
    User ubuntu
    IdentityFile ~/.ssh/ubuntu
```

配置好之后，你只需要执行 `ssh ubuntu` 就可以登录服务器。

## 2. 配置 Docker Context

Docker Contexts 是 Docker 19.03 版本引入的新特性，它封装了连接到不同 Docker 环境所需的所有配置，用于更方便地管理多个 Docker 主机（无论是本地还是远程）。

如果你用过 Kubernetes，可以把它理解为 `kubectl config` 中的 `context`—— 它能让你在不同 Kubernetes 集群间无缝切换。Docker Context 也类似，只不过它切换的是 Docker 守护进程。

创建一个指向远程服务器的 Context 非常简单：

```shell
# docker context -create <context名称> --docker "host=ssh://<服务器别名> --description "<描述>"
# 或者 docker context -create <context名称> --docker "host=ssh://<服务器用户>@<服务器IP>" --description "<描述>"

docker context create remote-ubuntu --docker "host=ssh://ubuntu" --description "Home Ubuntu server"
```

创建成功后，可以使用 `docker context ls` 查看当前 Context 列表：

```shell
$ docker context ls                                            

NAME            DESCRIPTION                               DOCKER ENDPOINT                             ERROR
default *       Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                 
desktop-linux   Docker Desktop                            unix:///Users/xxxx/.docker/run/docker.sock   

remote-ubuntu   Home Ubuntu server                        ssh://ubuntu
```

可以看到，我们有一个本地 `default` Context 和一个刚创建的 `remote-ubuntu` Context。名称后的星号表示当前正在使用的 Context。

## 3. 使用远端 Docker

创建好 Context 之后，执行以下命令，将 Docker 环境切换到远程服务器：

```shell
# docker context use <context名称>
docker context use remote-ubuntu
```

切换过后，你就可以与远端的 `dockerd` 通信了——你执行的每个 `docker` 命令都会通过 SSH 隧道，以远程的服务器上执行。当你需要在远程服务器上构建 x86 镜像，或者运行 container 时，都可以直接执行 `docker build`、`docker run`。

简单验证一下：

```shell
$ docker info | grep "Operating System"
Operating System: Ubuntu 22.04.5 LTS
```

如果想切回本地的 Docker 环境，也很简单：

```shell
docker context use default
```

## 局限性

虽然通过 SSH + Docker Context 可以使用远端 Docker，在某些场景下存在局限性。最主要的两点是本地文件挂载和端口映射的行为与我们的直觉不符。

### 1. 文件挂载

当你在本地执行 `docker run -v /local/path:/container/path my-image` 时，你可能会期望将本地的 `/local/path` 目录挂载到 container 中。但实际上，由于 `dockerd` 运行在远端服务器上，它会尝试挂载远端服务器上的 `/local/path` 目录。这意味着我们无法将本机的目录挂载到 container 中，基本告别使用远端 Docker + [dev container](https://code.visualstudio.com/docs/devcontainers/containers) 开发了。

### 2. 端口映射

类似地，当你使用 `-p` 参数（例如 `docker run -p 8080:80 my-nginx`）时，端口 `8080` 会被映射到远端服务器的网络接口上，而不是你的本地主机。


### 3. 构建上下文（Build Context）

执行 `docker build .` 时，Docker 客户端会将当前目录（构建上下文）完整地打包，并通过 SSH 发送到远端服务器。如果你的项目包含大量文件，或者你的网络连接不佳，这个上传过程可能会非常缓慢。

## 延伸阅读

- [Configure remote access for Docker daemon](https://docs.docker.com/engine/daemon/remote-access/)
- [Protect the Docker daemon socket](https://docs.docker.com/engine/security/protect-access/)