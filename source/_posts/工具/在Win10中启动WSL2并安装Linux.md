---
title: 在 Win10 中启动 WSL2 并安装 Linux
categories:
  - 工具
tags:
  - Docker
  - WSL2
date: 2022-05-27 22:41:07
description: 听公司大佬安利的 WSL2，提升开发姿势水平。记录一次安装 WSL2 的过程。
---

本文内容：

1. 了解  WSL 和 WSL2
2. 在 Windows 10 上开启 WSL2 并安装 Ubuntu
3. 安装 docker

参考官网文档： https://docs.microsoft.com/zh-cn/windows/wsl/

## 什么是 WSL

`WSL`，即 `Windows Subsystem for Linux`，适用于 Linux 的 Windows 子系统可让开发人员按原样运行 GNU/Linux 环境 (包括大多数命令行工具、实用工具和应用程序)， 且不会产生虚拟机开销。

您可以：

- [在 Microsoft Store](https://aka.ms/wslstore) 中选择你偏好的 GNU/Linux 分发版。
- 运行常用的命令行软件工具（例如 `grep`、`sed`、`awk`）或其他 ELF-64 二进制文件。
- 运行 Bash shell 脚本和 GNU/Linux 命令行应用程序，包括：
  - 工具：vim、emacs、tmux
  - 语言：[NodeJS](https://docs.microsoft.com/zh-CN/windows/nodejs/setup-on-wsl2)、Javascript、[Python](https://docs.microsoft.com/zh-CN/windows/python/web-frameworks)、Ruby、C/C++、C# 与 F#、Rust、Go 等
  - 服务：SSHD、[MySQL](https://docs.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-database)、Apache、lighttpd、[MongoDB](https://docs.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-database)、[PostgreSQL](https://docs.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-database)。
- 使用自己的 GNU/Linux 分发包管理器安装其他软件。
- 使用类似于 Unix 的命令行 shell 调用 Windows 应用程序。
- 在 Windows 上调用 GNU/Linux 应用程序。

## 什么是WSL2

WSL 2 是适用于 Linux 的 Windows 子系统体系结构的一个新版本，它支持适用于 Linux 的 Windows 子系统在 Windows 上运行 ELF64 Linux 二进制文件。 它的主要目标是**提高文件系统性能**，以及添加**完全的系统调用兼容性**。

这一新的体系结构改变了这些 Linux 二进制文件与Windows 和计算机硬件进行交互的方式，但仍然提供与 WSL 1（当前广泛可用的版本）中相同的用户体验。

单个 Linux 分发版可以在 WSL 1 或 WSL 2 体系结构中运行。 每个分发版可随时升级或降级，并且你可以并行运行 WSL 1 和 WSL 2 分发版。 WSL 2 使用全新的体系结构，该体系结构受益于运行真正的 Linux 内核。

总结如下：

1. WSL 2 是 WSL 中体系结构的新版本，它更改 Linux 发行版与 Windows 交互的方式。
2. WSL 2 的主要目标是提高文件系统性能并增加系统调用的完全兼容性。 
3. 每个 Linux 发行版都可以作为 WSL 1 或 WSL 2 发行版运行，并可随时进行切换。 
4. WSL 2 是底层体系结构的主要功能，它使用虚拟化技术和 Linux 内核来实现其新功能。

## WSL 1 和 WSL 2 功能比较

![image-20220414221049850](http://image.kongxiao.top/image-20220414221049850.png)

## 安装 WSL

详细安装过程可参考文档 https://docs.microsoft.com/zh-cn/windows/wsl/install

> **一步安装指令**： `wsl --install`
>
> 在管理员 PowerShell 或 Windows 命令提示符中输入此命令，然后重启计算机

此命令将：

- 启用可选的 WSL 和虚拟机平台组件
- 下载并安装最新 Linux 内核
- 将 WSL 2 设置为默认值
- 下载并安装 Ubuntu Linux 发行版（可能需要重新启动）

**旧版的 Windows 可能不支持这种方式，所以需要进行手动分步安装，如下所述。**


### 步骤 1 - 启用适用于 Linux 的 Windows 子系统

以 **管理员身份** 打开 `PowerShell`

![image-20220414230052575](http://image.kongxiao.top/image-20220414230052575.png)

输入如下命令 (然后根据提示重启电脑)：


```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

![image-20210330232921672](http://image.kongxiao.top/image-20210330232921672.png)

或者也可以：

设置 → 应用和功能 → 右侧相关设置中的程序和功能 → 左侧启用或关闭`windows`功能：

勾选启动的`Linux`的`windows`子系统这个选项，确定后重启电脑。

![image-20220414230645657](http://image.kongxiao.top/image-20220414230645657.png)

### 步骤 2 - 检查运行 WSL 2 的要求

更新到 WSL 2，需要运行 Windows 10。

- 对于 x64 系统：**版本 1903** 或更高版本，采用 **内部版本 18362** 或更高版本。

>要检查 Windows 版本及内部版本号，选择 Windows 徽标键 + R，然后键入“winver”，选择“确定”

### 步骤 3 - 启用虚拟机功能

安装 WSL 2 之前，必须启用“虚拟机平台”可选功能。 计算机需要[虚拟化功能](https://docs.microsoft.com/zh-cn/windows/wsl/troubleshooting#error-0x80370102-the-virtual-machine-could-not-be-started-because-a-required-feature-is-not-installed)才能使用此功能。

以管理员身份打开 PowerShell 并运行：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

**重新启动** 计算机，以完成 WSL 安装并更新到 WSL 2。

这部也可以在启用或关闭`windows`功能菜单中设置。

### 步骤 4 - 下载 Linux 内核更新包 

下载地址：[适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

下载完成后运行上一步中下载的更新包

### 步骤 5 - 将 WSL 2 设置为默认版本

打开 PowerShell，然后在安装新的 Linux 发行版时运行以下命令，将 WSL 2 设置为默认版本：

```powershell
wsl --set-default-version 2
```

### 步骤 6 - 安装所选的 Linux 分发

打开 [Microsoft Store](https://aka.ms/wslstore)，搜索 Ubuntu，选择"获取"

![image-20220427220957231](http://image.kongxiao.top/image-20220427220957231.png)

安装完成后启动新安装的 Linux 分发版，创建用户帐户和密码即可使用。

![image-20220427231116456](http://image.kongxiao.top/image-20220427231116456.png)

### 步骤 7 - 安装 Windows 终端

参考文档：https://docs.microsoft.com/zh-CN/windows/terminal/install

在微软应用商城获取 **Windows Terminal**

![image-20220414232541281](http://image.kongxiao.top/image-20220414232541281.png)

## WSL2 与 Windows 之间共享文件

由于 WSL2 为 Windows 原生支持的子系统所以无需设置即可直接共享目录等。

1. Windows 中直接访问 WSL2 中的文件或文件夹
   在 wsl 中输入如下命令，使用资源管理器 Explorer 打开当前的目录。

```shell
explorer.exe .
```

![image-20220414233148418](http://image.kongxiao.top/image-20220414233148418.png)

或者访问路径 \\wsl$\Ubuntu-20.04

![image-20220427231234276](http://image.kongxiao.top/image-20220427231234276.png)

2. 在 WSL2 中所有的磁盘已被挂载到了 Linux发行版中/mnt 目录下，在此目录下默认会把 Windows 中所有已挂载的盘也映射到 /mnt目录下：

![image-20220414233401466](http://image.kongxiao.top/image-20220414233401466.png)

默认的 wsl 没有设置 root 密码，需要通过如下指令设置：

```sh
sudo  passwd root
```

## 安装 Docker

更换 Ubuntu 官方源为国内源，加快下载速度

```bash
vim /etc/apt/sources.list
```

将内容替换为下方：

```shell
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal universe
deb http://mirrors.aliyun.com/ubuntu/ focal-updates universe
deb http://mirrors.aliyun.com/ubuntu/ focal multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-security universe
deb http://mirrors.aliyun.com/ubuntu/ focal-security multiverse
```

接下来添加 Docker 源，依次执行如下命令：

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```shell
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

```

```shell
sudo apt update
```

配置完成软件源之后安装 Docker 并启动：

```shell
sudo apt install -y docker-ce
```

```shell
sudo service docker start
```

启动后如果不 sudo 执行 Docker 的命令会提示没有权限：

![image-20220522205636231](http://image.kongxiao.top/20220522205644.png)

通过将用户添加到[docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020)用户组可以将 sudo 去掉命令如下：

```shell
sudo groupadd docker #添加docker用户组

sudo gpasswd -a $USER docker #将登陆用户加入到docker用户组中

newgrp docker #更新用户组
```



好了，现在可以愉快的在 windows 机器上玩 docker 了。