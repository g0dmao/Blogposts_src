---
title: WSL下连接USB设备
tags:
  - Linux
  - USB
  - WSL2
categories:
  - Linux
  - WSL2
abbrlink: 13715
date: 2025-06-11 23:49:53
---
<details>

<summary>版权信息</summary>

:::warning

本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。

:::

</details>

---

⭐⭐⭐本文参考自微软官方WSL文档——[连接 USB 设备 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb)

由于WSL不提供本机连接USB设备的支持，因此需要安装开源项目usbipd-win来支持USB的共享连接。

## 什么是usbpid？
usbipd 是一个**用于管理 USB/IP（USB over IP）服务的命令行工具**，可以在 Windows 下使用。 USB/IP 是一种协议，允许**通过网络共享USB 设备**。 usbipd工具允许用户在 Windows 上共享 USB 设备，使其他计算机能够通过网络访问这些设备。

## 安装 USBIPD-WIN 项目
1. 转到 [usbipd-win 项目的最新发布页面](https://github.com/dorssel/usbipd-win/releases)。
2. 选择 .msi 文件，该文件将下载安装程序。 （你可能会收到一条警告，要求你确认你信任此下载）。
3. 运行下载 usbipd-win_x.msi 安装程序文件。

这将安装：
- 名为 `usbipd` 的服务，（显示名称：USBIP 设备主机）。 可以使用 Windows 中的服务应用检查此服务的状态。
- 命令行工具 `usbipd`。 此工具的位置将添加到 PATH 环境变量。
- 名为 `usbipd` 的防火墙规则，用于允许所有本地子网连接到服务。 可修改此防火墙规则以微调访问控制。

4. 若要附加 USB 设备，请运行以下命令。 （不再需要使用提升的管理员提示。确保 WSL 命令提示符处于打开状态，以使 WSL 2 轻型 VM 保持活动状态。 **请注意，只要 USB 设备连接到 WSL，Windows 将无法使用它。** 一旦连接到 WSL，任何在 WSL 2 上运行的发行版都可以使用该 USB 设备。 请确认设备是否已连接 `usbipd list`。 在 WSL 提示符下，运行 `lsusb` 以验证 USB 设备是否已列出，并且可以使用 Linux 工具与之交互。

```powershell
usbipd attach --wsl --busid <busid>
```

5. 打开 Ubuntu（或首选 WSL 命令行），并使用以下命令列出附加的 USB 设备：

```bash
lsusb
或者
lsblk -f
```

若没有找到命令，则先下载`usbutils`
![](Snipaste_2025-06-12_00-11-23.png)
```bash
sudo apt install usbutils
```

可以看到已经成功读取到u盘，并且能够使用普通 Linux 工具与之交互。 根据应用程序，可能需要配置 udev 规则，以允许非根用户访问设备。
![](Snipaste_2025-06-12_00-18-59.png)
6. 在 WSL 中使用设备后，可以物理断开 USB 设备的连接，或者从 PowerShell 运行以下命令：

```powershell
usbipd detach --busid <busid>
```


