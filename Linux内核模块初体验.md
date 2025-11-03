---
title: Linux内核模块初体验
layout: post
copyright: "false"
abbrlink: 49513
date: 2025-09-27 15:48:35
tags:
  - 内核
  - 内核编程
categories:
  - Linux
  - 内核编程
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---


>本篇摘自野火嵌入式Linux系列教程驱动开发篇


## 内核模块基本概念

### 现代内核派系

单内核：关键功能和服务功能均在内核空间提供

 - 运行效率高
 - 扩展性较差

微内核：内核空间只提供关键功能，服务功能在用户空间提供

- 运行效率较低
- 安全性、扩展性较高

linux属于单内核，按理来说扩展性不好，但其的模块化设计又弥补了这一点。
### 内核模块加载/卸载

- 使用insmod命令加载
- 使用rmmod命令卸载

### 内核模块入口/出口

- module_init()：加载模块式该函数自动执行，进行初始化操作
- module_exit()：卸载模块时函数自动执行，进行清理操作

### 内核模块信息声明

- MODULE_LICENSE()：表示模块代码接受的软件许可协议，Linux内核遵循GPL V2开源协议，内核模块与linux内核保持一致即可。
- MODULE_AUTHOR()：描述模块的作者信息
- MODULE_DESCRIPTION()：对模块的简单介绍
- MODULE_ALIAS()：给模块设置一个别名

## 内核模块实验1

### 实验环境

- 开发板烧录好Debian镜像。
- SSH连接MobaXterm
- 启动开发板，搭建好SFTP。
- 获取Debian镜像的内核源码并编译。

### 编译4.19.71版本内核

内核模块的功能需要依赖内核提供的各种底层接口

1.下载linux内核源码

​	github:

```
git clone https://github.com/Embedfire/ebf-buster-linux.git
```

​	gitee:

```
git clone https://gitee.com/Embedfire/ebf-buster-linux.git
```

2.安装必要环境工具库

```
sudo apt install make gcc-arm-linux-gnueabihf gcc bison flex libssl-dev dpkg-dev lzop
```

- gcc-arm-linux-gnueabihf 交叉编译器
- bison 语法分析器
- flex 词法分析器
- libssl-dev OpenSSL通用库
- lzop LZO压缩库的压缩软件

3.一键编译内核

```
sudo ./make_deb.sh
```

>如果在电脑上编译不成功且无法解决，可以选择在开发板上编译，代价就是。。。嗯 it takes a long time

4.获取编译出来的内核相关文件

```
YOURPATH/ebf_linux_kernel_6ull_depth1/build_image/
```

### 内核模块头文件

- `#include <linux/module.h>`：包含内核模块信息声明的相关函数
- `#include <linux/init.h>`：包含了 module_init()和 module_exit()函数的声明
-  `#include <linux/kernel.h>`：包含内核提供的各种函数，如printk

### 内核模块打印函数

- `printf`：glibc实现的打印函数，工作于用户空间，不可在内核使用

- `printk`：内核模块无法使用glibc库函数，内核自身实现的一个类printf函数，但是需要指定打印等级。
  - `#define KERN_EMERG` 	"<0>" 通常是系统崩溃前的信息
  - `#define KERN_ALERT`          "<1>" 需要立即处理的消息
  - `#define KERN_CRIT`             "<2>" 严重情况
  - `#define KERN_ERR`              "<3>" 错误情况
  - `#define KERN_WARNING`      "<4>" 有问题的情况
  - `#define KERN_NOTICE`       "<5>" 注意信息
  - `#define KERN_INFO`            "<6>" 普通消息
  - `#define KERN_DEBUG`        "<7>" 调试信息

查看当前系统printk打印等级：`cat /proc/sys/kernel/printk`
输出
```
7       4       1       7
```
表示：
- 当前控制台日志级别
- 默认消息日志级别
- 最小的控制台级别
- 默认控制台日志级别

小于等于设定打印等级的消息不会被打印。
打印内核所有打印信息：dmesg

- 内核log缓冲区大小有限制，缓冲区数据可能被冲掉

### 源码展示

```c
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/module.h>
static int __init hello_init(void){
    printk(KERN_EMERG"[KERN_EMERG]Hello Kernel!\n");
    printk("[DEFAULT]Hello Kernel!\n");
    return 0;
}

static void __exit hello_exit(void){
    printk("[DEFAULT]Goodbye Kernel!\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("MIT");
MODULE_AUTHOR("gdm");
MODULE_DESCRIPTION("print: hello kernel");
MODULE_ALIAS("test_module");
```

### Makefile分析

```makefile
# 定义内核路径
KERNEL_DIR:=/home/debian/linux/driver_learning/ebf_linux_kernel_6ull_depth1/build_image/build
# 定义编译工具集
ARCH:=arm
CROSS_COMPILE:=arm-linux-gnueabihf-
# 将变量导出，相当于作为环境变量，让内核Makefile继承此变量 否则变量只在当前文件可见。
export ARCH CROSS_COMPILE
# 内核模块编译的标准变量，告诉内核编译哪个模块。
obj-m:=hellokernal.o
# -C 命令指切换到目标目录的makefile
# M= 可以理解为module，指定模块所在路径
# modules 是内核makefile定义的目标，触发“构建外部模块”完整规则
all:
    $(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) moudules
.PHONY:clean
clean:
    $(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean
```

- `KERNEL_DIR`：指向linux内核具体路径
- `export`：导出变量给子Makefile使用 
- `obj-m` := <模块名>.o：定义要生成的模块
- `$(MAKE)`：Makefile的默认变量，值为make
- `选项”-C”`：让make工具跳转到linux内核目录下读取顶层Makefile
- `M=`：表示内核模块源码目录
- `$(CURDIR)`：Makefile默认变量，值为当前目录所在路径
- `make modules`：执行Linux顶层Makefile的伪目标，它实现内核模块的源码读取并编译为.ko文件

### 编译内核模块

```
make
```

### 开发板加载内核模块

```
insmod xxx.ko
```


