# 快速开始

*我们假设你已经熟悉 Prime 的基础操作，并且用 HP 连接套件传输过几个原生程序。*

## 目录

1. [准备工作](#1-准备工作)
2. [编译官方示例](#2-编译官方示例)
3. [下一步](#3-下一步)

## 1. 准备工作

### 计算器

你有一台 Prime 计算器吗？有？很好，你已经在 99% 的人前面了！

不过，要利用 Nbprime SDK，请对你的计算器做以下几点检查：

**硬件型号**：我们目前只支持 G1。你可以在计算器的“关于 HP Prime”界面看到信息：V1 的硬件版本为 A，V2 的硬件版本为 C。

> **为什么不支持 G2？**
>
> G2 使用的操作系统与 G1（V1 和 V2）不同，我们无法利用官方的后门漏洞。

**固件版本**：同样在“关于 HP Prime”界面：`Software Build Date` 是否为 `20250915`？这是当前唯一支持的版本。如果不是，下面会详细指导你将固件升级为 `20250915` 版本。

#### 刷新固件

你需要：

- 一台 Prime G1 计算器；
- 一台 PC（或 Mac），安装了 HP 连接套件（HP Connectivity Kit）；
- 一根 Micro-USB 数据线（**普通充电线是不行的！** 这样会识别不到 Prime）；
- `20250915` 固件，预装后门。

**详细操作**：

1. 将开机状态下的 Prime 连接至 PC；
2. 打开连接套件，右键你的 Prime 的图标，选择“更新固件”；
3. 耐心等待：不要动数据线、计算器及连接套件！否则有变砖风险！

> 另外，如果是全新的 Prime，最好先装上电池并充电。

### PC

我们希望你有一台 PC。在移动设备上用终端模拟器也是可行的（如同一位核心开发者一样），但是你可能无法在出现问题时获取适合移动设备的解决方案。

我们目前只支持 Linux 系统或 WSL2。Windows 开发只需对 toolchain 稍作修改，但是官方目前不提供此类指导。接下来的操作以 Debian/Ubuntu（或对应 WSL2 子系统）为例：

1. 确保已安装 `gcc`、`make`、`python3` 和 `binutils`；若没有，可用 `apt install` 安装；
2. 克隆 `toolchain` 仓库：

   ```bash
   git clone https://github.com/nbprime-team/toolchain.git
   cd toolchain
   ```

3. 引导安装：

   ```bash
   make bootstrap
   ```

4. 验证工具链并导出：

   ```bash
   make verify
   source scripts/env.sh
   ```

真不错！这就是你需要的全部准备工作。

## 2. 编译官方示例

### 了解仓库

如果你认真阅读了 [`README.md`](https://github.com/nbprime-team/.github/profile/README.md)，你应该大致了解组织的构造了。不过，还是从头概览一下仓库吧。

核心仓库：

- `.github`：这里是项目的门面；
- `toolchain`：这是你构建程序或项目时所需要的核心工具链，与一些 PC 端辅助工具；
- `docs`：这里是人工文档，涵盖了大致的指导，但不包括项目内容。我们保证你不会看到这里有**任何 AI 生成的东西**。

项目仓库：

- `app-collection`：这是收集二进制小应用的仓库；这些应用大多单文件、小体积，而且应用文件夹里包含了预编译产出（而不是单独创建 releases 并管理版本）。有一些应用虽然不是二进制，但是属于二进制开发中 Prime 端的实用程序。我们的示例构建从这里开始；
- `legacy`：这是一些原生平台程序（MicroPython / PPL）；它们对于项目是有参考或移植价值的，但是仅用于存放，**不做任何版本或稳定性保证**；
- `Prime-xxx`：这是大型的二进制项目，依赖多、体积大。项目内含源码和依赖，预编译分发版在 releases 页面，分不同版本。

### 编译示例

`app-collection/examples` 专用于存放示例：理论上，你通过阅读它们即可了解标准 C 库的接口及调用方法。

> 标准 C 库是 toolchain 里的一些文件，提供了基础的 API/ABI 接口。它的目标是保持精简实用而非覆盖所有功能。

我们先把 git 仓库拉下来：

```bash
git clone https://github.com/nbprime-team/app-collection.git
cd app-collection
```

如果你 `cd examples/suika`，你会看到里面有一个 `suika.hpappdir` 文件夹：这是预编译，你可以直接将其上传 Prime 运行。但是我们要了解构建的方法，尤其是当你希望调整示例或开发自己的程序时。

在这里，编译极为简单：

```bash
make -C examples/suika deploy
```

现在 `suika.hpappdir` 更新了（如果你修改了示例，这个就是修改后的版本）。你可以用连接套件或 `msd` 应用程序传至 Prime 上，主程序页面会有 `suika` 图标，双击运行。

## 3. 下一步

你成功了吗？好！现在你具有了开发二进制程序的全部条件，转到 [`developing`](https://github.com/nbprime-team/docs/developing.md) 去，那里你将拥有自己的二进制程序。
