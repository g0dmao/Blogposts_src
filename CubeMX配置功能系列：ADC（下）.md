---
title: CubeMX配置功能系列：ADC（下）
abbrlink: 55933
date: 2025-03-09 20:01:13
tags:
  - STM32
  - CubeMX
  - ADC
  - 嵌入式
categories:
  - 嵌入式
  - CubeMX配置功能
description: 本篇介绍了STM32 ADC的相关配置。
---
<details>

<summary>版权信息</summary>

:::warning

本文章遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。

:::

</details>

---

## 目录
- [目录](#目录)
- [1. 输入模式](#1-输入模式)
  - [1.1. 单端输入](#11-单端输入)
  - [1.2. 差分输入](#12-差分输入)
- [2. 继续看config部分](#2-继续看config部分)
  - [2.1. 通用设置部分：](#21-通用设置部分)
  - [2.2. ADC 设置部分：](#22-adc-设置部分)
    - [2.2.1. ClockPre...](#221-clockpre)
    - [2.2.2. Resolusion](#222-resolusion)
    - [2.2.3. Data Ali...](#223-data-ali)
    - [2.2.4. Gain Com...](#224-gain-com)
    - [2.2.5. Scan Conversion Mode](#225-scan-conversion-mode)
    - [2.2.6. End Of Conversion Selection](#226-end-of-conversion-selection)
    - [2.2.7. Low Power Auto Wait](#227-low-powerautowait)
    - [2.2.8. Continuous Conversion Mode](#228-continuous-conversion-mode)
    - [2.2.9. Discontinuous Conversion Mode](#229-discontinuous-conversion-mode)
    - [2.2.10. DMA Continuous Requests](#2210-dma-continuous-requests)
    - [2.2.11. Overrun behaviour](#2211-overrun-behaviour)
    - [2.2.12. Conversion Data Managerment Mode（H7）](#2212-conversion-data-managerment-modeh7)
  - [2.3. 规则转换模式](#23-规则转换模式)
    - [2.3.1. Enable Regular Conversions](#231-enable-regular-conversions)
    - [2.3.2. Enable Regular Oversamping（不常用）](#232-enable-regular-oversamping不常用)
    - [2.3.3. Oversamping Right Shift（不常用）](#233-oversamping-right-shift不常用)
    - [2.3.4. Number Of Conversion](#234-number-of-conversion)
    - [2.3.5. External Trigger Conversion Soure（触发转换的外部来源）](#235-external-trigger-conversion-soure触发转换的外部来源)
    - [2.3.6. External Trigger Conversion Edge（触发转换的外部沿）](#236-external-trigger-conversion-edge触发转换的外部沿)
    - [2.3.7. Rank](#237-rank)
    - [2.3.8. Offset Number](#238-offset-number)
  - [2.4. 然后是注入转换模式](#24-然后是注入转换模式)
    - [2.4.1. 规则通道：](#241-规则通道)
    - [2.4.2. 注入通道：](#242-注入通道)
  - [2.5. 最后是可爱的看门小狗🥰~](#25-最后是可爱的看门小狗)
- [3. 参考函数](#3-参考函数)
- [4. 参考博客](#4-参考博客)


## 1. 输入模式
我们打开cubemx的配置，配置通道时有如下选择：分别是差分输入和单端输入。
![](Snipaste_2025-03-09_19-30-43.png)

### 1.1. 单端输入

单端输入方式优点就是简单，缺点是如果VIN受到干扰，由于GND电位始终是0V，所以最终采样值也会随着干扰而变化。采样值=VIN（叠加干扰值 ）- 0V

如图所示，单端输入只有一个输入引脚ADCIN，使用公共地GND作为电路的返回端，ADC的采样值=ADCIN电压-GND的电压(0V)。这种输入方式优点就是简单，缺点是如果vin受到干扰，由于GND电位始终是0V，所以最终ADC的采样值也会随着干扰而变化。
![](8c60645887a08582fd0cd8f9cba6b8da.png)

### 1.2. 差分输入

差分受到的干扰是差不多的，输入的共模干扰，在输入时会被减掉，从而降低了干扰，缺点就是接线复杂一些。

差分信号 优点：易分辨小信号、抗干扰EMS强；缺点:双线

而差分输入比单端输入多了一根线，最终的ADC采样值=(ADCIN电压)-(ADCIN-电压)，由于通常这两根差分线会布在一起，所以他们受到的干扰是差不多的，输入共模干扰，在输入ADC时会被减掉，从而降低了干扰，缺点就是接线复杂一些。而且需要VIN+和VIN-两路反相的输入信号。

差分输入的是将两个输入端的差值作为信号，这样可以免去一些误差，比如你输入一个1V的信号可电源有偏差实际输入要大0.1.就可以用差分输入1V和2V一减就把两端共有的那0.1误差剪掉了。单端输入无法去除这类误差。

![](Snipaste_2025-03-09_19-43-28.png)

## 2. 继续看config部分

![](Snipaste_2025-03-09_19-45-21.png)
![](Snipaste_2025-03-09_19-45-32.png)

### 2.1. 通用设置部分：
- 当只启用1个ADC时，只能选择**独立模式**，如果使用双ADC并要求同步的话，则会有更多选择。双重ADC同步模式，两个ADC同时采集一个或多个通道，可以提高采样率。这个双ADC同步模式的选择先不作记录了！
### 2.2. ADC 设置部分：
#### 2.2.1. ClockPre...
时钟预分频，双时钟域架构。目的是让ADC达到稳定的工作频率[^1]（异步时钟模式（Asynchronous clock mode，基于PLL2P时钟）同步时钟模式（Synchronous clock mode，基于AHB时钟）有些型号的单片机则是直接从时钟树专门分出了一个ADC时钟频率配置。
![](Snipaste_2025-03-09_20-30-53.png)

#### 2.2.2. Resolusion
分辨率。不再赘诉。请见上篇

#### 2.2.3. Data Ali...
数据对齐。不再赘诉。请见上篇

#### 2.2.4. Gain Com... 
增益补偿：对所有转换后的数据进行增益补偿。每次转换后，数据根据增益补偿对采样后的数据进行变换。

#### 2.2.5. Scan Conversion Mode
扫描模式。不再赘诉。请见上篇

#### 2.2.6. End Of Conversion Selection
转换结束标志选择：指定转换结束后是否产生EOS中断或单次转换结束事件标志（EOC）。有End of single conversion（EOC） 与 End of sequence of conversion（EOS）两种选择。这两个事件会触发中断与DMA。\
\
在多通道转换过程中，如果选择了End of sequence of conversion，会在一组数据转换完成后发出EOS标志，如下图所示。如果不选，则不会置位该标志。
![](Snipaste_2025-03-09_20-42-09.png)
选择EOS目的是等所有通道转换完毕后，产生中断后将全部数据取出来，或者使用DMA将全部数据取出来。

#### 2.2.7. Low Power Auto Wait
低功耗自动延迟等待模式，可选参数为 ENABLE 和 DISABLE，当使能时，仅当一组内所有之前的数据已处理完毕时，才开始新的转换，适用于低频应用。**该模式仅用于 ADC 的轮询模式，不可用于 DMA 以及中断。**

#### 2.2.8. Continuous Conversion Mode
是否启用连续转换。不再赘诉。请见上篇 **若想使用ADC+DMA的话，必须先使能连续转换模式**。

#### 2.2.9. Discontinuous Conversion Mode
间断模式，再赘诉一下。这里的不连续含义是指每次触发进行一个子组的转换，跟Continuous Conversion Mode的连续含义不一样。例如使能了该配置，该参数的下方就立马出现Number Of Discontinuous Conversions，如果它设为2，且ADC1使能了通道1，2，5，7，10，11的话，那么第一次触发ADC1采样时，就会采样通道1与通道2的值，再一次触发ADC1采样的话，就会采样通道5与通道7值，如此类推。值得注意的是，Continuous Conversion Mode与Discontinuous Conversion Mode不能同时使能，两者不能共存。

#### 2.2.10. DMA Continuous Requests
DMA连续请求：指定 DMA 请求是否以一次性模式执行(当达到转换次数时，DMA 传输停止)或在连续模式下(DMA 传输无限制，无论转换的数量)。[^2]
- 不使能：在这种模式下，每次有新的转换数据可用时，ADC都会生成一个DMA传输请求，一旦DMA到达最后一个DMA传输，即使转换已经再次开始，ADC也会停止生成DMA请求，适合转换固定数量数据的情况。
- 使能：在这种模式下，每当数据寄存器中有新的转换数据可用时，即使DMA已经到达最后一次DMA传输，ADC也会生成DMA传输请求。这允许在循环模式下配置DMA来处理连续的模拟输入数据流。

#### 2.2.11. Overrun behaviour
溢出处理：用于配置ADC转换数据未及时读取，造成溢出时的处理。
- Overrun data preserved:保留旧数据，丢弃和丢失新的转换。
- Overrun data overwritten:数据寄存器被最后一次转换的结果覆盖，之前未读的数据丢失
![](Snipaste_2025-03-09_21-17-16.png)

#### 2.2.12. Conversion Data Managerment Mode（H7）
转换数据管理模式。不使用DMA的话，不使用DFSDM数字滤波器做后期处理的话，选择Regular Conversion data stored in DR register only即可。其实就是选择存放转换完成的模拟量数据的地方而已。

### 2.3. 规则转换模式

#### 2.3.1. Enable Regular Conversions
使能规则转换。使能它才能采集各个通道上的模拟量。

#### 2.3.2. Enable Regular Oversamping（不常用）
使能规则过采样。
![](Snipaste_2025-03-10_14-20-12.png)

#### 2.3.3. Oversamping Right Shift（不常用）
过采样右移。**过采样器能将累加的采样值进行右移**。有什么用？比如过采样设置15，那么将采集16个值进行累加。接着配置右移动4位的话，相当于将刚才的累加值除以16，得到平均值。不需要在程序里求平均了。当然，如果大家喜欢在程序里求平均值也是可以的。

#### 2.3.4. Number Of Conversion
转换通道数。根据ADC配置的通道数来选择。有多少转换通道就设置几。

#### 2.3.5. External Trigger Conversion Soure（触发转换的外部来源）
选择触发转换的来源：
- Regular Conversion launched by software （软件触发）
- Timer 1 Capture Compare 1 event （定时器1捕获比较事件1）
- … （各种定时器触发来源）
一般使用软件触发就行。

#### 2.3.6. External Trigger Conversion Edge（触发转换的外部沿）
选择定时器触发时，需要进一步选择触发的沿。选择软件触发时，该项为None。
Trigger detection on the rising edge（上升沿）
Trigger detection on the falling edge（下降沿）
Trigger detection on the rising and falling edge（上升与下降沿）

#### 2.3.7. Rank
![](Snipaste_2025-03-10_14-29-51.png)

- Channel：选择采样的通道。
-  Sampling Time：采样时间。配置多少个时钟周期，建议采样时间尽量长一点，以获得较高的准确度。总的转换时间=采样时间+逐次逼近时间（TSAR）。[^3]
逐次逼近参考时间：请查阅相关数据手册。
这是一个参考：
![](Snipaste_2025-03-10_14-31-52.png)

#### 2.3.8. Offset Number
偏移序号？？不懂。

### 2.4. 然后是注入转换模式
我们看到，在选择了ADC的相关通道引脚之后，在模拟至数字转换器中有两个通道：注入通道与规则通道。规则通道至多16个，注入通道至多4个。

#### 2.4.1. 规则通道：
规则通道相当于你正常运行的程序，看它的名字就可以知道，很规矩，就是正常执行程序。
#### 2.4.2. 注入通道：
注入通道可以打断规则通道，听它的名字就知道不安分，如果在规则通道转换过程中，有注入通道进行转换，那么就要先转换完注入通道，等注入通道转换完成后，再回到规则通道的转换流程。类似于中断。
![](Snipaste_2025-03-10_14-43-22.png)

### 2.5. 最后是可爱的看门小狗🥰~
当被ADC转换的模拟电压值低于低阈值或高于高阈值时，便会产生中断。阈值的高低值由ADC_LTR和ADC_HTR配置。
可以防止读取到的电压值超量程或者低于量程，也可以监控CPU温度，防止温度过高，反正是用处多多。

## 3. 参考函数
```c
//开启ADC 3种模式 ( 轮询模式 中断模式 DMA模式 )
HAL_ADC_Start(&hadcx);       //轮询模式开启ADC
HAL_ADC_Start_IT(&hadcx);       //中断轮询模式开启ADC
HAL_ADC_Start_DMA(&hadcx);       //DMA模式开启ADC

//关闭ADC 3种模式 ( 轮询模式 中断模式 DMA模式 )
HAL_ADC_Stop();
HAL_ADC_Stop_IT();
HAL_ADC_Stop_DMA();

//ADC校准函数 ：F4系列不支持
HAL_ADCEx_Calibration_Start(&hadcx);      

//读取ADC转换值
HAL_ADC_GetValue();

//等待转换结束函数.第一个参数为哪个ADC,第二个参数为最大等待时间
HAL_ADC_PollForConversion(&hadc1, 50);

//ADC中断回调函数
HAL_ADC_ConvCpltCallback();

//转换完成后回调，DMA模式下DMA传输完成后调用

//规则通道及看门狗配置
HAL_ADC_ConfigChannel(); //配置规则组通道
HAL_ADC_AnalogWDGConfig();
```



## 4. 参考博客
有星星的是我觉得写的很好的博客。
- ⭐⭐⭐[STM32H743-梳理ADC模数转换器在CubeMX上的配置_overrun behaviour-CSDN博客](https://blog.csdn.net/wallace89/article/details/117048846)遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议
- ⭐⭐⭐[STM32CubeIde ADC配置详解_end of conversion selection-CSDN博客](https://blog.csdn.net/demonneverhunts/article/details/135155881)遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议
- ⭐⭐⭐[【STM32】HAL库 STM32CubeMX教程九---ADC_cubemx adc-CSDN博客](https://blog.csdn.net/as480133937/article/details/99627062)
- [ADC的单端输入、伪差分输入、差分输入区别？_差分adc-CSDN博客](https://blog.csdn.net/chenhuanqiangnihao/article/details/122086308)


[^1]:ADC的工作频率需要查阅相关数据手册,工作频率太高会导致转换无法完成。
[^2]: 在连续模式下，DMA 必须配置为循环模式。否则，当达到 DMA 缓冲区最大指针时将触发溢出。
[^3]: 时间可以用ADC时钟周期来衡量。例如12bitADC的逐次逼近时间为12.5个ADC时钟周期
