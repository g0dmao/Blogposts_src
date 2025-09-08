---
title: 浅析IMX镜像文件结构
date: 2025-07-14 19:24:15
tags:
  - Linux
  - boot
categories:
  - Linux
  - 记录
layout: draft
---
<details>

<summary>版权信息</summary>

:::warning

本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。

:::

</details>

---

:::info
学习笔记记录，非教程。
:::

## 当使用内部BootROM启动时，发生了什么？

当我们通过设置BOOT_MODE[1:0]为内部启动模式时，即芯片通过执行内部的bootROM固有的代码来启动，在此模式下，芯片会执行内部的 boot ROM 代码，这段 boot ROM 代码会进行硬件初始化(一部分外设)，然后从 boot 设备(就是存放代码的设备、比如 SD/EMMC、NAND)中将代码拷贝出来复制到指定的 RAM 中，一般是 DDR。

- 设置内核时钟为396MHz。使能MMU和Cache，使能L1cache、L2 cache、MMU，目的就为了加速启动。
- 从BOOT_CFG设置的外置存储中，读取image，然后做相应的处理。

![](Snipaste_2025-07-14_19-59-23.png)![](Snipaste_2025-07-14_20-01-17.png)
其中内存管理单元（Memory Management Unit）用于将物理地址翻译为虚拟内存地址，以及通过虚拟内存访问实际物理内存。而初始化高速缓存则是加速数据访问。

## 镜像文件结构

在上一节[【点灯大师】点亮I.MX6ULL开发板的LED灯](https://blog.godmao.top/posts/43400/)提到过，通过汇编生成的bin文件并不能直接使用，还需要添加额外的头文件信息。我们使用VSCODE（需要扩展hex-editor）打开生成的img文件，这个文件是上一节我们使用`mkimage.sh`生成的。我们可以先查看一下这个sh文件方便理解：
我们只截取通过sd卡启动的配置参数：
```shell
elif [ "$1" == "sd" ]; then
    ../bin/$IMG_BUILDER --combine base_addr=0x80000000 ivt_offset=0x400 app_offset=0x2000 dcd_file=dcd.bin app_file=sdk20-app.bin ofile=sdk20-app.img image_entry_point=0x80002000
```

可以看到，该脚本为我们默认设置的基址为0x80000000，也就是绝对的起始地址，在此基础上，偏移app_offset即为应用程序的起始地址。因此程序的起始地址（bin的起始地址）为0x80002000，ivt表则是偏移了1024字节，0x400。
![](Snipaste_2025-07-14_21-25-46.png)
接下来我们看img文件的内容：
![](Snipaste_2025-07-14_21-34-13.png)