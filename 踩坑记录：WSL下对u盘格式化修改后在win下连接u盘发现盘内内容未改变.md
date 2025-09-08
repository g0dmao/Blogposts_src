---
title: 踩坑记录：WSL下对u盘格式化修改后在win下连接u盘发现盘内内容未改变
tags:
  - Linux
  - 踩坑记录
categories:
  - Linux
  - 记录
abbrlink: 20676
date: 2025-06-14 18:15:30
---
<details>

<summary>版权信息</summary>

:::warning

本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。

:::

</details>

---
## 问题描述
我将USB读卡器插入microSD卡（该SD卡此前刷入过micropython固件包），并将其连接至电脑，接着我在win系统下，在SD卡目录下创建了一个文件wtf.txt，然后打开WSL，使用usbip将该u盘共享，在WSL下，u盘能正确读取、挂载，我在WSL重新对这个u盘使用fdisk重新分区，mkfs重新格式化为FAT32文件系统，并且在WSL下touch了一个新文件hi.txt，使用sync同步，然后使用usbipd的命令断开连接，接入win系统后，wtf.txt居然还在，再次将u盘接入WSL，发现hi.txt也在，也就是说，同样的SD卡，两个系统读到的东西却不一样，这是怎么回事？
按照网上的主流方法，尝试过编译内核添加usb驱动支持，手动安装usbip工具都不管用。
## 问题解决
在使用fdisk分区时，我发现如下警告：
```text
The device contains hybrid MBR -- writing GPT only.
```

在询问chatgpt后，弄清了这条警告的含义：
1. **磁盘中检测到 Hybrid MBR**：即使之前格式化或重分区过，它仍可能保留有旧的混合结构。
2. **当前正尝试用 GPT 格式重写该磁盘**：工具会忽略 MBR 部分，只写入纯 GPT。    

**Hybrid MBR（混合主引导记录）** 是一种兼容机制，它允许一个磁盘同时拥有：
- 一部分 **MBR（传统 BIOS 启动支持）**
- 一部分 **GPT（用于现代系统的分区方案）**

这种结构通常用于让 **使用 GPT 分区的磁盘**仍然可以在一些只识别 MBR 的系统中启动，比如旧版 BIOS 系统。

我突然想起来这个SD卡不就是之前用于引导启动micropython固件的吗，所以存在混合结构，而正是这样的混合结构，是使得在win下，系统读到的是MBR分区表，Linux下读到的是GPT分区表，这就像 SD 卡前面贴着两张标签纸（MBR 和 GPT）：
- Windows 看标签 A，就看到旧文件
- Linux 看标签 B，就看到新文件
- 它们根本没在“同一张纸”上操作

于是我按照GPT提供的方法，使用命令彻底清除磁盘前部（包括 MBR & GPT）：
```bash
sudo dd if=/dev/zero of=/dev/sdd bs=1M count=10
sync
```
完全抹除了旧的结构信息，再次重复分区、格式化、挂载、创建文件等操作，这下两个系统读取到的文件就同步了😍！

## 特别感谢

- [ChatGPT](https://chatgpt.com/)!，AI改变世界啊~
- 我自己！
