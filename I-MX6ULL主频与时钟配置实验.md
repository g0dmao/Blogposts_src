---
title: I-MX6ULL主频与时钟配置实验
abbrlink: 33477
date: 2025-08-06 21:50:03
tags:
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


## 系统时钟浅析

### 7路PLL

为了方便生成时钟，6从24MHz晶振生出来7路PLL。这7路PLL中有的又生出来PFD。
- `PLL1`：ARM PLL供给ARM内核。
- `PLL2`：sysytem PLL，528MHz，528_PLL，此路PLL分出了4路PFD，分别为PLL2_PFD0~PFD3
- `PLL3`: USB1 PLL，480MHz 480_PLL，此路PLL分出了4路PFD，分别为PLL3_PFD0~PFD3。
- `PLL4`: Audio PLL，主供音频使用。
- `PLL5`: Video PLL，主供视频外设，比如RGB LCD接口，和图像处理有关的外设。
- `PLL6`：ENET PLL，主供网络外设。
- `PLL7`: USB2_PLL ,480MHz，无PFD。

详见 IMX6ULL参考手册 Chapter 18 Clock Controller Module (CCM)

附 时钟树图：
t形为多路选择器

![](Snipaste_2025-08-06_22-02-07.png)
![](Snipaste_2025-08-06_22-03-45.png)


### 要初始化的PLL和PFD

- PLL1，
- PLL2，以及PLL2_PFD0~PFD3.
- PLL3以及PLL3_PFD0~PFD3.
其他需要时再设置。
一般按照时钟树里面的值进行设置。

## 具体配置

### 系统主频的配置

根据时钟树来设定系统主频。具体时钟配置的说明可以参见 IMX6ULL参考手册 18.5.1.5
CCM internal clock generation 
下图为时钟切换器 （CCM_CLK_SWITCHER） 控制输出的示意图。

![](Snipaste_2025-08-07_10-00-08.png)

1. 要设置ARM内核主频为528MHz，设置CACRR寄存器的ARM_PODF位为2分频，然后设置PLL1=1056MHz即可。CACRR的bit3~0为ARM_PODF位，可设置0~7，分别对应1~8分频。应该设置CACRR寄存器的ARM_PODF=1。

2. 切换时钟源。PLL1输出为pll1_sw_clk。pll1_sw_clk有两路可以选择，分别为pll1_main_clk，和step_clk，通过CCSR寄存器的pll1_sw_clk_sel位(bit2)来选择。为0的时候选择pll1_main_clk，为1的时候选额step_clk。

3. 在修改PLL1的时候，也就是设置系统时钟的时候需要给6ULL一个临时的时钟，也就是step_clk。在修改PLL1的时候需要将pll1_sw_clk切换到step_clk上。

4. 设置step_clk。Step_clk也有两路来源，由CCSR的step_sel位(bit8)来设置，为0的时候设置step_clk为osc=24MHz。为1的时候不重要，不用。

5. 时钟切换成功以后就可以修改PLL1的值。

6. 通过CCM_ANALOG_PLL_ARM寄存器的DIV_SELECT位(bit6~0)来设置PLL1的频率，公式为：

		Output = fref*DIV_SEL/2  1056=24*DIV_SEL/2=>DIEV_SEL=88。
		
		设置CCM_ANALOG_PLL_ARM寄存器的DIV_SELECT位=88即可。PLL1=1056MHz
		
		还要设置CCM_ANALOG_PLL_ARM寄存器的ENABLE位(bit13)为1，也就是使能输出。

7. 在切换回PLL1之前，设置置CACRR寄存器的ARM_PODF=1！！切记。

### 其他PLL设置

PLL2和PLL3。PLL2固定为528MHz，PLL3固定为480MHz。

1. 初始化PLL2_PFD0~PFD3。寄存器CCM_ANALOG_PFD_528用于设置4路PFD的时钟。比如PFD0= 528 * 18 / PFD0_FRAC。 设置PFD0_FRAC位即可。比如PLL2_PFD0=352M=528 *  18 / PFD0_FRAC，因此FPD0_FRAC=27。
2. 同理初始化PLL3_PFD0~PFD3

## 外设时钟

AHB_CLK_ROOT、PERCLK_CLK_ROOT以及IPG_CLK_ROOT。

因为PERCLK_CLK_ROOT和IPG_CLK_ROOT要用到AHB_CLK_ROOT，所以我们要初始化AHB_CLK_ROOT。

### AHB_CLK_ROOT的初始化

AHB_CLK_ROOT=132MHz。

设置CBCMR寄存器的PRE_PERIPH_CLK_SEL位，设置CBCDR寄存器的PERIPH_CLK_SEL位0。设置CBCDR寄存器的AHB_PODF位为2，也就是3分频，因此396/3=132MHz。

### IPG_CLK_ROOT初始化

设置CBCDR寄存器IPG_PODF=1，也就是2分频。

### PERCLK_CLK_ROOT初始化

设置CSCMR1寄存器的PERCLK_CLK_SEL位为0，表示PERCLK的时钟源为IPG。


## 这篇文章写得很不错,摘录一些

[【嵌入式Linux】i.MX6ULL 时钟树——理论分析-CSDN博客](https://blog.csdn.net/Beihai_Van/article/details/139868239)

### 总结PFD配置寄存器:

CCM_ANALOG_PFD_528n 寄存器控制着 i.MX 6ULL 处理器中四个分数分频器的配置，包括时钟门控、稳定性状态和分数分频值。这些分频器用于生成不同频率的时钟信号，以满足各种外设的需求。

1. CLKGATE 位 (Clock Gate): 时钟门控
- 作用: CLKGATE 位控制着对应 PFD 的时钟信号是否被开启或关闭。
- 值:
	- 0: 时钟信号开启，PFD 正常工作，可以输出分频后的时钟信号。
	- 1: 时钟信号关闭，PFD 处于关闭状态，不输出时钟信号。
- 目的:
	- 省电: 当某个外设不需要时钟信号时，可以通过设置 CLKGATE 位为 1 来关闭该 PFD 的时钟，从而减少功耗。
	- 控制时钟信号: 在某些情况下，可能需要动态地控制某个外设的时钟信号，例如在系统启动或进入低功耗模式时。
2. STABLE 位 (Stable): 稳定性状态

- 作用: STABLE 位指示对应 PFD 的输出时钟信号是否已经稳定。
- 值:
	当新的分数分频值生效时，位域的值会反转（从 0 变为 1 或从 1 变为 0）。这个反转就像一个信号，表明分频器已经完成调整。
- 目的:
	- 诊断: STABLE 位是一个只读位，用于诊断 PFD 的稳定性。
	- 确保时钟信号质量: 在修改 PFD 的分数分频值后，需要等待 STABLE 位反转，才能确保输出的时钟信号稳定可靠。

### i.MX6U 芯片 PLL 时钟详解

i.MX6U 芯片拥有多个 PLL（Phase-Locked Loop，锁相环）模块，用于生成各种频率的时钟信号，为芯片内部的不同模块和外设提供时钟源。下面整理了 i.MX6U 芯片的 7 个主要 PLL：

1. ARM_PLL (PLL1)
	- 用途: 为 ARM 内核提供时钟信号。
	- 倍频: 可编程，最高可倍频至 1.3GHz。
	- 特点: ARM 内核的运行速度直接取决于此 PLL 的输出频率。
2. 528_PLL (PLL2)
	- 用途: 为系统总线、内部逻辑单元、DDR 接口、NAND/NOR 接口等提供时钟源。
	- 倍频: 固定 22 倍频，不可编程。
	- 输出频率: 24MHz * 22 = 528MHz。
	- 特点: 该 PLL 以及其生成的 4 路 PFD (PLL2_PFD0~PLL2_PFD3) 是 i.MX6U 内部系统总线的核心时钟源。
3. USB1_PLL (PLL3)
	- 用途: 主要用于 USB1PHY，但也可作为其他外设的时钟源。
	- 倍频: 固定 20 倍频。
	- 输出频率: 24MHz * 20 = 480MHz。
	- 特点: 该 PLL 以及其生成的 4 路 PFD (PLL3_PFD0~PLL3_PFD3) 可用于多种外设。
4. USB2_PLL (PLL7)
	- 用途: 为 USB2PHY 提供时钟信号。
	- 倍频: 固定 20 倍频。
	- 输出频率: 24MHz * 20 = 480MHz。
	- 特点: 虽然序号标为 4，但实际是 PLL7。
5. ENET_PLL (PLL6)
	- 用途: 用于生成网络所需的时钟信号。
	- 倍频: 固定 20+5/6 倍频。
	- 输出频率: 24MHz * (20+5/6) = 500MHz。
	- 特点: 可在此 PLL 的基础上生成 25/50/100/125MHz 的网络时钟。
6. VIDEO_PLL (PLL5)
	- 用途: 用于显示相关外设，例如 LCD。
	- 倍频: 可调整，输出范围在 650MHz~1300MHz。
	- 分频: 可选 1/2/4/8/16 分频。
	- 特点: 可根据显示设备的需求调整输出频率和分频比。
7. AUDIO_PLL (PLL4)
	- 用途: 用于音频相关外设。
	- 倍频: 可调整，输出范围在 650MHz~1300MHz。
	- 分频: 可选 1/2/4 分频。
	- 特点: 可根据音频设备的需求调整输出频率和分频比。
总结:

i.MX6U 芯片通过多个 PLL 模块，生成各种频率的时钟信号，为芯片内部的不同模块和外设提供时钟源。每个 PLL 的倍频和分频都可以根据需要进行配置，以满足不同外设的需求。
