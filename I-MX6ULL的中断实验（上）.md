---
title: I-MX6ULL的中断实验（上）
tags:
  - 嵌入式
  - Linux
categories:
  - Linux
  - 记录
description: 本篇学习CortexA7中断系统和配置中断的启动文件的编写。
abbrlink: 32440
date: 2025-08-11 21:17:21
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

- [1. Cortex-A7中断系统](#1-cortex-a7中断系统)
  - [1.1. Cortex-A中断向量表](#11-cortex-a中断向量表)
  - [1.2. 中断向量表偏移](#12-中断向量表偏移)
  - [1.3. GIC v2中断控制器](#13-gic-v2中断控制器)
    - [1.3.1. GIC 的主要功能](#131-gic-的主要功能)
    - [1.3.2. 核心结构](#132-核心结构)
    - [1.3.3. 核心原理（简单版）](#133-核心原理简单版)
  - [1.4. 中断号](#14-中断号)
- [2. 添加中断向量表](#2-添加中断向量表)
- [3. 复位中断函数的编写](#3-复位中断函数的编写)
  - [3.1. 操作CP15协处理器](#31-操作cp15协处理器)
  - [3.2. 中断向量表偏移](#32-中断向量表偏移)
  - [3.3. 设置sp指针](#33-设置sp指针)
  - [3.4. 清除.bss段](#34-清除bss段)
- [4. IRQ中断函数的编写](#4-irq中断函数的编写)
  - [4.1. 保存中断现场（防止破坏原任务状态）](#41-保存中断现场防止破坏原任务状态)
  - [4.2. 从 GIC（通用中断控制器）读取中断号](#42-从-gic通用中断控制器读取中断号)
  - [4.3. 切换到 SVC 模式执行 C 语言中断处理函数](#43-切换到-svc-模式执行-c-语言中断处理函数)
  - [4.4. 结束中断并恢复现场](#44-结束中断并恢复现场)
- [5. 整个启动文件](#5-整个启动文件)


## 1. Cortex-A7中断系统

### 1.1. Cortex-A中断向量表
Cortex-A中断向量[^1]表有8个中断，其中重点关注IRQ。Cortex-A的中断向量表需要用户自己去定义。

![](Snipaste_2025-08-11_21-20-27.png)

各中断的简单介绍：
![](Snipaste_2025-08-11_21-27-07.png)

其中IRQ中断为非向量中断，所有中断共享同一入口，再软件判断来源。
大概是这样：
![](Snipaste_2025-08-11_21-27-26.png)

### 1.2. 中断向量表偏移

通过设置中断向量表偏移，指定中断向量表的地址。

### 1.3. GIC v2中断控制器

类似于STM32的NVIC，CortexA7使用 GIC v2 作为中断控制器，比NVIC更强大因为GIC能处理多核的中断。
#### 1.3.1. GIC 的主要功能

1. **收集中断**  
    接收来自外设和 CPU 内部的各种中断请求（SPI、PPI、SGI）。
2. **优先级管理**  
    按优先级裁定哪个中断先被处理。
3. **目标选择**  
    把中断发给指定的 CPU 核，或广播给多个核。
4. **中断屏蔽与使能**  
    软件可选择屏蔽某个中断或所有中断。
5. **确认与结束**  
    CPU 核心在处理前向 GIC 确认（acknowledge），处理完向 GIC 报告结束（end of interrupt）。
#### 1.3.2. 核心结构

ARM GIC 主要由两部分组成：
- **Distributor（分发器）**  
    负责接收所有外设中断，决定送到哪一个 CPU。
- **CPU Interface（CPU 接口）**  
    每个 CPU 核对应一个接口，负责和 Distributor 通信，接收/确认/结束中断。

#### 1.3.3. 核心原理（简单版）

GIC 的核心流程基本就是三步：
1. **接收中断请求**  
    外设（SPI）、特定 CPU 相关的中断（PPI）、核间中断（SGI）都送进 GIC。  
2. **仲裁优先级 + 选择目标 CPU**  
    - 比较中断优先级（Priority）        
    - 检查中断是否被屏蔽（Enable Mask）    
    - 根据目标 CPU 配置（Target CPU Mask）决定发给谁    
3. **通知 CPU → CPU 响应 → 完成汇报**
    - 发信号到目标 CPU 接口 
    - CPU 读取中断号（acknowledge）并执行 ISR   
    - ISR 结束后 CPU 通知 GIC（end of interrupt）

### 1.4. 中断号
中断的 ID Card。
![](Snipaste_2025-08-11_21-54-28.png)


## 2. 添加中断向量表

- **向量表的“表项”是指令，不是纯地址。**
    
- 每个向量入口的指令都指向某个具体的处理函数（Reset_Handler、IRQ_Handler 等）。
    
- 当发生异常，CPU 硬件会取指执行入口指令 → `ldr pc, =xxx` → 跳到真正的处理函数。
    

这样，**只要依次写好这些跳转指令，后续再告诉cpu这些指令在哪个位置，就等于告诉 CPU“各种异常该去哪”**。

:::danger
**中断向量表必须按固定顺序来定义**，顺序是由 ARM 硬件架构规定的，不是随便排列的。这样CPU才能正确处理不同的异常
:::

```asm
/* 设置中断向量表，当中断来时，CPU就会执行对应指令 */

    /* 顺序不能改变，名字可以改变！ */

    ldr pc, =Reset_Handler      /* 复位中断                     */  

    ldr pc, =Undefined_Handler  /* 未定义中断                    */

    ldr pc, =SVC_Handler        /* SVC(Supervisor)中断        */

    ldr pc, =PrefAbort_Handler  /* 预取终止中断                   */

    ldr pc, =DataAbort_Handler  /* 数据终止中断                   */

    ldr pc, =NotUsed_Handler    /* 未使用中断                    */

    ldr pc, =IRQ_Handler        /* IRQ中断                    */

    ldr pc, =FIQ_Handler        /* FIQ(快速中断)未定义中断           */
```

## 3. 复位中断函数的编写

### 3.1. 操作CP15协处理器

**关闭I、D Cache 和MMU**

<details>

<summary>为什么要这样做？</summary>

CP15 是 Cortex-A 系列处理器的 **系统控制寄存器集**，其中 **SCTLR（System Control Register, c1）** 控制了处理器的一些核心行为，例如：

| 位   | 功能                              |
| --- | ------------------------------- |
| M   | MMU 使能位（Memory Management Unit） |
| C   | 数据缓存使能位                         |
| I   | 指令缓存使能位                         |
| A   | 对齐检查（Alignment）使能               |
| Z   | 缓存清零 / 压缩乘法指令                   |
| V   | 高速异常向量表                         |
| …   | 其他一些调试、异常行为控制                   |

在复位后，SCTLR 寄存器的初始值不一定是你想要的运行状态。不同的 SoC 或板级支持包可能默认值不同，有些位可能 **默认启用某些功能但不适合裸机启动阶段**。

在裸机启动阶段，你通常需要：

1. **禁用 MMU 和缓存**：
    
    - 在初始化页表和内存映射前，启用 MMU 会导致访问非法地址或产生未定义行为。
    - 数据缓存如果没有正确初始化，可能会导致数据写入内存不一致。
        
2. **关闭对齐检查**：
    
    - 对齐检查在裸机启动阶段可能会导致异常，因为启动代码可能使用非对齐访问。
        
3. **保证指令执行顺序一致**：
    
    - SCTLR 的某些位会影响指令缓存和流水线行为。
    - 在系统初始化阶段，确保缓存和 MMU关闭可以让你更容易调试和保证代码执行顺序。
        
4. **统一系统行为**：
    
    - 不同芯片可能在复位后 SCTLR 寄存器的默认值不同。
    - 显式初始化可以保证启动代码在各种芯片上行为一致。

如果不重置这些位可能的风险：

- **访问未定义**：
    
    - 如果 MMU 默认启用但页表未初始化，访问内存可能触发数据异常。
        
- **数据不一致**：
    
    - 缓存启用但未正确设置缓存策略，读写内存可能得到错误数据。
        
- **异常频发**：
    
    - 对齐检查默认启用，而启动代码中可能有非对齐访问，导致异常。
        
- **调试困难**：
    
    - 程序执行行为不确定，难以定位启动问题。


</details>

代码如下：
```asm
* 禁用MMU、cache、对齐检查等，配置适合裸机启动的环境，这样的启动代码移植性更好 */

    mrc     p15, 0, r0, c1, c0, 0     /*  读取CP15系统控制寄存器   */

    bic     r0,  r0, #0x1000          /*  清除第12位（I位）禁用 I Cache  */

    bic     r0,  r0, #0x4             /*  清除第 2位（C位）禁用 D Cache  */

    bic     r0,  r0, #0x2             /*  清除第 1位（A位）禁止严格对齐   */

    bic     r0,  r0, #0x800           /*  清除第11位（Z位）分支预测   */

    bic     r0,  r0, #0x1             /*  清除第 0位（M位）禁用 MMU   */

    mcr     p15, 0, r0, c1, c0, 0     /*  将修改后的值写回CP15寄存器   */
```

### 3.2. 中断向量表偏移

访问CP15 VBAR（Vector Base Address Register）寄存器，该寄存器是专门指定中断向量表偏移首地址的。
- `dsb`（Data Synchronization Barrier）保证所有数据访问指令执行完再继续。
- `isb`（Instruction Synchronization Barrier）保证新的 VBAR 设置马上生效，不会被 CPU 指令流水线里的旧指令影响。

```asm
/* 访问CP15 VBAR寄存器 设置中断向量表偏移 */

    ldr r0, =0x80000000

    dsb

    isb /* 数据同步指令 */

    mcr p15, 0, r0, c12, c0, 0

    dsb

    isb
```

### 3.3. 设置sp指针

内核工作在不同模式下，User模式和是Sys模式共用sp寄存器，而其他模式是独享一个sp寄存器。
- ARMv7-A 每个异常模式都有自己的栈指针寄存器，如果不设置，进入该模式时可能破坏未知内存。
- 给每种模式单独分配内存空间，可以防止中断嵌套、函数调用等操作时栈互相冲突。


### 3.4. 清除.bss段

<details>

<summary>bss段是什么？</summary>

在编译链接后，程序的内存布局通常分为几个主要段：

- **.text**：存放代码（只读）。
    
- **.data**：存放已初始化的全局/静态变量（初始值不为 0）。
    
- **.bss**：存放**未初始化的**或者**初始化为 0** 的全局/静态变量。

`.bss` 只在运行时需要分配内存，编译产物中不占用存储空间（只是记录了大小），这样可以减少可执行文件体积。

</details>

<details>

<summary>为什么要这样做？</summary>

根据 **C 语言标准**：

> 所有未显式初始化的全局变量、静态变量在程序开始执行前必须被初始化为 0。

这意味着：

- 如果 `bss` 段中的内容不是全 0，程序可能会读到**随机的旧内存内容**。
    
- 硬件上，RAM 启动后可能是**杂乱的值**（上电状态、上次运行残留、调试器写入等）。
    

因此，**启动代码（crt0 或汇编 startup.s）通常会在 `main()` 之前清零 bss 段**，保证所有这些变量是干净的 0。

</details>

```asm
/* 清除bss段 防止未初始化变量等数据访问错误 */

    ldr r0, =__bss_start

    ldr r1, =__bss_end

    mov r2, #0

bss_clear:          /* ia: increse after */

    stmia r0!, {r2} /* stmia: 从r0所存地址开始，将r2的值写入该地址，写入后地址自动递增 */

    cmp r0, r1

    ble bss_clear /* less & equal */
```


## 4. IRQ中断函数的编写


<details>

<summary>中断函数实现了什么？（详细版）</summary>

### 4.1. 保存中断现场（防止破坏原任务状态）

一旦 CPU 进入 IRQ 模式，中断处理代码首先：

- 把 `lr`（中断返回地址）、`r0`~`r3`、`r12` 等易被破坏的寄存器压栈保存。
    
- 保存 `SPSR`（中断进入时的 CPSR 状态），以便中断结束时恢复原任务的运行状态。
    

**目的**：防止中断服务过程破坏被中断的程序的寄存器内容和运行状态。

---

### 4.2. 从 GIC（通用中断控制器）读取中断号

- 读取 **GICC_IAR**（Interrupt Acknowledge Register）寄存器。
    
- 这个寄存器会返回当前触发的中断 ID（是哪一个外设触发的 IRQ）。
    

**目的**：确定是哪个具体中断源发生了 IRQ。

---

### 4.3. 切换到 SVC 模式执行 C 语言中断处理函数

- ARMv7-A 的 IRQ 模式不适合直接运行通用 C 代码（栈、寄存器不统一），所以这里切到 SVC 模式。
    
- 在 SVC 模式下调用 `system_irqhandler`（C 写的总中断处理函数）。
    
- `system_irqhandler` 会根据中断号去调用具体外设的中断处理例程。
    

**目的**：统一用 SVC 模式的栈和环境执行中断逻辑，方便用 C 语言写 ISR 分发器。

---

### 4.4. 结束中断并恢复现场

- 切回 IRQ 模式。
    
- 向 GIC 写 **EOIR**（End of Interrupt Register），通知 GIC 这个中断处理完成，可以接收新的中断。
    
- 从栈中弹出寄存器和 `SPSR`，恢复进入中断前的状态。
    
- 用 `subs pc, lr, #4` 返回到中断发生前的指令位置继续运行。
    

**目的**：

- 告诉中断控制器“我处理完了”。
    
- 让 CPU 完整回到被打断的程序，像中断没发生一样继续执行。

</details>

省流版：
当一个程序在 **SVC 模式** 下运行时，如果来了 IRQ：
1. CPU 自动切到 IRQ 模式。
2. 把 **SVC 模式的 CPSR** 存进 **IRQ 模式的 SPSR**。
3. 把返回地址放进 IRQ 模式的 LR。
4. 开始执行 IRQ 模式的向量入口代码（也就是 `IRQ_Handler`）。
irq_handler:
- 保存现场
- 查中断号
- 切换到svc模式 调用用c语言编写的中断分发函数 根据中断号分发到中断处理函数 来执行中断处理
- 结束中断、恢复现场
ps： 最后一行subs指令有点抽象。。。。。。。。


## 5. 整个启动文件

```asm
.global _start
/* 以下两个变量在链接脚本中定义 */
.global __bss_start
.global __bss_end
_start:
    @ /* 1.设置处理器模式为SVC模式(其实CotexA内核上电默认即为SVC模式故不需要写) */
    @ mrs r0, cpsr                /* 读取cpsr到r0 */
    @ bic r0, r0, #0x1f           /* bic bit-clear位清零 */          
    @                             /* 等同于R0 = R0 & (~Operand2) 这个操作数2自己推算*/
    @                             /* 现在这个操作相当于将r0的前5位清零了 */
    @ orr r0, r0, #0x13           /* orr 按位或 */
    @ msr cpsr, r0                /* 写入cpsr */  
    /* 设置中断向量表，当中断来时，CPU就会执行对应指令 */
    /* 顺序不能改变，名字可以改变！ */
    ldr pc, =Reset_Handler      /* 复位中断                     */  
    ldr pc, =Undefined_Handler  /* 未定义中断                    */
    ldr pc, =SVC_Handler        /* SVC(Supervisor)中断        */
    ldr pc, =PrefAbort_Handler  /* 预取终止中断                   */
    ldr pc, =DataAbort_Handler  /* 数据终止中断                   */
    ldr pc, =NotUsed_Handler    /* 未使用中断                    */
    ldr pc, =IRQ_Handler        /* IRQ中断                    */
    ldr pc, =FIQ_Handler        /* FIQ(快速中断)未定义中断           */
Reset_Handler:  
    cpsid i                           /* Change Processor State Interrupt or abort Disable irq */

    /* 禁用MMU、cache、对齐检查等，配置适合裸机启动的环境，这样的启动代码移植性更好 */
    mrc     p15, 0, r0, c1, c0, 0     /*  读取CP15系统控制寄存器   */
    bic     r0,  r0, #0x1000          /*  清除第12位（I位）禁用 I Cache  */
    bic     r0,  r0, #0x4             /*  清除第 2位（C位）禁用 D Cache  */
    bic     r0,  r0, #0x2             /*  清除第 1位（A位）禁止严格对齐   */
    bic     r0,  r0, #0x800           /*  清除第11位（Z位）分支预测   */
    bic     r0,  r0, #0x1             /*  清除第 0位（M位）禁用 MMU   */
    mcr     p15, 0, r0, c1, c0, 0     /*  将修改后的值写回CP15寄存器   */
    /* 访问CP15 VBAR寄存器 设置中断向量表偏移 */
    ldr r0, =0x80000000
    dsb
    isb /* 数据同步指令 */
    mcr p15, 0, r0, c12, c0, 0
    dsb
    isb
    /* 清除bss段 防止未初始化变量等数据访问错误 */
    ldr r0, =__bss_start
    ldr r1, =__bss_end
    mov r2, #0
bss_clear:          /* ia: increse after */
    stmia r0!, {r2} /* stmia: 从r0所存地址开始，将r2的值写入该地址，写入后地址自动递增 */
    cmp r0, r1
    ble bss_clear /* less & equal */  
    /* 设置各个模式下的栈指针，
     * 注意：IMX6UL的堆栈是向下增长的！
     * 堆栈指针地址一定要是4字节地址对齐的！！！
     * DDR范围:0X80000000~0X9FFFFFFF
     */
    /* 进入IRQ模式 */
    mrs r0, cpsr
    bic r0, r0, #0x1f   /* 将r0寄存器中的低5位清零，也就是cpsr的M0~M4  */
    orr r0, r0, #0x12   /* r0或上0x13,表示使用IRQ模式                   */
    msr cpsr, r0        /* 将r0 的数据写入到cpsr_c中                    */
    ldr sp, =0x80600000 /* 设置IRQ模式下的栈首地址为0X80600000,大小为2MB */
    /* 进入SYS模式 */
    mrs r0, cpsr
    bic r0, r0, #0x1f   /* 将r0寄存器中的低5位清零，也就是cpsr的M0~M4  */
    orr r0, r0, #0x1f   /* r0或上0x13,表示使用SYS模式                   */
    msr cpsr, r0        /* 将r0 的数据写入到cpsr_c中                    */
    ldr sp, =0x80500000 /* 设置SYS模式下的栈首地址为0X80400000,大小为2MB */
    /* 进入SVC模式 */
    mrs r0, cpsr
    bic r0, r0, #0x1f   /* 将r0寄存器中的低5位清零，也就是cpsr的M0~M4  */
    orr r0, r0, #0x13   /* r0或上0x13,表示使用SVC模式                   */
    msr cpsr, r0        /* 将r0 的数据写入到cpsr_c中                    */
    ldr sp, =0X80400000 /* 设置SVC模式下的栈首地址为0X80200000,大小为2MB */
    cpsie i             /* 打开全局中断 */
    /* 跳转到main函数 */
    b main
/* 未定义中断 */
Undefined_Handler:
    ldr r0, =Undefined_Handler
    bx r0
/* SVC中断 */
SVC_Handler:
    ldr r0, =SVC_Handler
    bx r0
/* 预取终止中断 */
PrefAbort_Handler:
    ldr r0, =PrefAbort_Handler  
    bx r0
/* 数据终止中断 */
DataAbort_Handler:
    ldr r0, =DataAbort_Handler
    bx r0  
/* 未使用的中断 */
NotUsed_Handler:
    ldr r0, =NotUsed_Handler
    bx r0  
/* IRQ中断！重点！！！！！ */
IRQ_Handler:
    /* 保护现场 */
    push {lr}                   /* 保存lr地址 */
    push {r0-r3, r12}           /* 保存r0-r3，r12寄存器 */
    mrs r0, spsr                /* 读取spsr寄存器 */
    push {r0}                   /* 保存spsr寄存器 */ 

    /* 读取GIC控制器基地址 */
    mrc p15, 4, r1, c15, c0, 0 /* 从CP15的C0寄存器内的值到R1寄存器中
                                * 参考文档ARM Cortex-A(armV7)编程手册V4.0.pdf P49
                                * Cortex-A7 Technical ReferenceManua.pdf P68 P138
                                */                          
    add r1, r1, #0X2000         /* GIC基地址加0X2000，也就是GIC的CPU接口端基地址 */
    /* 查询中断号 */
    ldr r0, [r1, #0XC]          /* GIC的CPU接口端基地址加0X0C就是GICC_IAR寄存器，
                                 * GICC_IAR寄存器保存这当前发生中断的中断号，我们要根据
                                 * 这个中断号来绝对调用哪个中断服务函数
                                 */
    push {r0, r1}               /* 保存r0,r1 */
    cps #0x13                   /* 进入SVC模式，允许其他中断再次进去 */
    push {lr}                   /* 保存SVC模式的lr寄存器 */
    ldr r2, =system_irqhandler  /* 加载C语言中断处理函数到r2寄存器中*/
    blx r2                      /* 运行C语言中断处理函数，带有一个参数，保存在R0寄存器中 */
    pop {lr}                    /* 执行完C语言中断服务函数，lr出栈 */
    cps #0x12                   /* 进入IRQ模式 */
    pop {r0, r1}                
    str r0, [r1, #0X10]         /* 中断执行完成，写EOIR，相当于确认中断执行完成 */
    pop {r0}                        
    msr spsr_cxsf, r0           /* 恢复spsr */
    pop {r0-r3, r12}            /* r0-r3,r12出栈 */
    pop {lr}                    /* lr出栈 */
    subs pc, lr, #4             /* 将lr-4赋给pc */
/* FIQ中断 */
FIQ_Handler:
    ldr r0, =FIQ_Handler    
    bx r0
loop:
    b loop
```


[^1]: 中断向量，英文名为Interrupt Vector，在早期计算机体系结构里，“vector”常被用作“指针/地址索引”的意思，所以”向量“实际上就是地址指针的意思。