---
title: Docker 部署笔记
date: 2024-06-29 23:48:19
tags: docker 部署 配置 
---
# Docker 部署笔记
## Docker简介
Docker 是一种广泛使用的容器化平台，可以在开发、测试和生产环境中轻松部署和管理应用程序。以下是从 Windows 和 Linux 系统上安装 Docker 的步骤及其原理的详细梳理。
## 老话，说说我为什么要掌握Docker?
Docker 已成为现代 IT 和开发环境中不可或缺的工具。对于我的职业发展和个人兴趣来说，掌握 Docker 具有显著的优势，尤其是在涉及技术环境部署、物联网、智能化设备以及 NAS 系统时。以下是我认为更详细的原因：
### 1. 职业发展需求：技术环境的部署和管理
    1.1. 高效的环境部署
    一致性：Docker 容器确保在开发、测试和生产环境中运行的应用程序具有一致性，无论底层环境如何变化。
    隔离性：每个容器独立运行，避免不同应用之间的冲突和依赖问题。
    快速启动：容器的轻量级特性使得启动速度非常快，便于快速部署和迭代。
    1.2. 易于扩展和管理
    弹性扩展：Docker 容器可以轻松地横向扩展，满足应用程序的负载需求。
    便捷的更新：通过镜像管理和版本控制，可以快速更新和回滚应用程序，简化维护工作。
    1.3. 强大的生态系统
    丰富的工具和服务：Docker 提供了从开发到部署的一整套工具，支持与 CI/CD 流水线、编排系统（如 Kubernetes）的无缝集成。
    社区支持：大量的开源项目和社区支持，使得 Docker 生态系统不断发展和完善。
### 2. 个人兴趣与物联网、智能化设备
    2.1. 快速试验和开发
    轻量化：Docker 容器可以在资源有限的设备上高效运行，适用于物联网和智能化设备的开发与部署。
    灵活性：无需修改底层操作系统，就可以在 Docker 容器中运行不同的服务和应用，方便测试和实验。
    2.2. 多样化的应用场景
    跨平台兼容性：Docker 支持在不同硬件架构上运行，适用于各种物联网设备和嵌入式系统。
    模块化：通过容器化，能将不同的功能模块化，便于管理和更新，提升开发效率。
### 3. NAS 系统中的最佳选择
    3.1. 轻松部署与管理
    简化复杂性：NAS（网络附加存储）设备通常用于家庭或小型办公室环境，Docker 可以简化这些设备上的应用部署过程。
    快速安装：通过 Docker，可以快速安装和配置各种常用的应用和服务，如文件共享、媒体服务器、数据库等。
    3.2. 高效资源利用
    低资源占用：相比传统的虚拟机，Docker 容器更加轻量，能更好地利用 NAS 设备的资源。
    并发运行：在同一个 NAS 上，可以同时运行多个 Docker 容器，满足多种应用需求。
    3.3. 灵活的扩展和升级
    动态调整：根据需要，随时可以增加或删除容器，灵活调整系统功能和容量。
    无缝升级：通过更新容器镜像，可以轻松完成系统和应用的升级，而无需重启整个 NAS 系统。

## Docker 原理
Docker 利用操作系统级别的虚拟化技术来提供软件容器，使得应用程序可以在隔离的环境中运行。我曾经探讨及查看，在宿主机中docker每一个启动容器都是一个现成一个信号，所以首先我们要了解主要的核心组件和概念包括：
### 1. Docker 镜像（Image）
定义：只读的模板，包含运行应用程序所需的一切，包括代码、库、环境变量等。[目前如果不科学上网，就多看看B站吧，从2024-06-07号开始，链路国外镜像源逐渐的不能用了
，后续再记录怎样替换镜像源，一言难尽]作用：用于创建 Docker 容器。
### 2. Docker 容器（Container）
定义：由镜像实例化而来的一个运行环境，是一个轻量级、独立的可执行包。
作用：确保应用程序及其依赖可以在任何环境下稳定运行。
### 3. Docker 引擎（Docker Engine） [Docker的软件及命令应用，在宿主机中会显示]
定义：Docker 的核心组件，包含 Docker 守护进程（Docker Daemon）、REST API 接口和命令行工具（CLI）。
作用：负责管理 Docker 容器和镜像。
### 4. Docker 文件（Dockerfile）[Docker既然是个容器，我可以通过这个方式构建脚本化生成我需要的容器运行逻辑]
定义：一个文本文件，包含一系列指令，用于定义如何构建 Docker 镜像。
作用：通过自动化的方式生成定制化的镜像。
### 5. Docker Compose  [配置文件一样的存在]
定义：用于定义和运行多容器 Docker 应用的工具。
作用：通过一个 YAML 文件来配置应用程序的服务、网络和卷。
二、Windows 系统上安装 Docker
1. 安装前的准备
系统要求：Windows 10 64-bit: Pro, Enterprise, or Education 版本，1903 或更高版本；或 Windows Server 2019。
硬件要求：启用硬件虚拟化（Intel VT-x 或 AMD-V）。
2. 安装步骤
2.1. 下载 Docker Desktop
访问[ Docker 官网](https://www.docker.com)下载安装包，mac操作系统直接下载dmg安装包。
下载适用于 Windows 的 Docker Desktop 安装程序。
2.2. 安装 Docker Desktop
双击安装包并按照提示完成安装。
在安装过程中，可以选择安装 WSL 2（适用于更高效的 Linux 容器运行）。
2.3. 配置 Docker Desktop
安装完成后，启动 Docker Desktop。
初次启动时，选择使用 WSL 2 还是 Hyper-V 作为后台支持。
完成基本配置，确保 Docker Desktop 已正常运行（图标应显示在系统托盘中）。
2.4. 验证安装
打开命令提示符或 PowerShell，运行以下命令验证 Docker 安装：
``` bash
docker --version  #显示 Docker 版本信息。
```
运行测试容器：
``` bash
docker run hello-world  #如果看到 "Hello from Docker!" 的输出，则说明 Docker 已成功安装和配置。
```
三、Linux 系统上安装 Docker
1. 安装前的准备
系统要求：现代的 Linux 发行版，通常建议使用最新的版本。
权限要求：需要具有 sudo 或 root 权限来安装和管理 Docker。
2. 安装步骤
2.1. 更新包管理器[Ubuntu apt管理器  ]
使用以下命令更新系统的包管理器（以 Ubuntu 为例）：
``` bash
sudo apt-get update  #如果看到 "Hello from Docker!" 的输出，则说明 Docker 已成功安装和配置。
```
2.2. 安装必要的依赖
安装依赖包，确保系统支持 HTTPS：
``` bash
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```
2.3. 添加 Docker GPG 密钥
使用以下命令添加 Docker 官方的 GPG 密钥：
``` bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
2.4. 添加 Docker 仓库
添加 Docker 的 APT 仓库：
``` bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
2.5. 安装 Docker CE
更新包管理器，并安装 Docker CE（Community Edition）：
``` bash
sudo apt-get update
sudo apt-get install docker-ce
```
2.6. 启动并验证 Docker
启动 Docker 服务：
``` bash
sudo systemctl start docker
```
验证 Docker 安装：
``` bash
sudo docker --version  #显示 Docker 版本信息。
```
运行测试容器：
``` bash
docker run hello-world  #如果看到 "Hello from Docker!" 的输出，则说明 Docker 已成功安装和配置。
```
2.7. （可选,但我一般都配置下方便，如果主机为业务机就不配置了）配置非 root 用户运行 Docker
创建 Docker 用户组，并将当前用户添加到组中：
``` bash
sudo groupadd docker
sudo usermod -aG docker $USER
```
退出并重新登录，以应用组更改。