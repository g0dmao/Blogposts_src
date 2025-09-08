---
title: MakeFile简明指南
layout: post
tags:
  - MakeFile
categories:
  - Linux
  - 项目管理
abbrlink: 4341
date: 2025-09-07 21:48:09
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

在软件开发中，项目通常包含很多源文件，如果每次编译都手动敲命令，不仅繁琐，还容易出错。  
**Makefile** 可以帮助我们自动化构建流程，大幅提升效率。本文将介绍 Makefile 的基础语法与常用用法。

---

- [1. 基础用法](#1-基础用法)
- [2. 执行逻辑^1](#2-执行逻辑1)
  - [2.1. 🔄 执行流程示意图](#21--执行流程示意图)
- [3. 伪目标 .PHONY](#3-伪目标-phony)
- [4. 变量](#4-变量)
  - [4.1. 赋值符号](#41-赋值符号)
- [5. 隐含规则与通配符](#5-隐含规则与通配符)
- [6. 条件分支](#6-条件分支)
  - [6.1. 语法](#61-语法)
  - [6.2. 示例一：根据平台选择clean执行方式](#62-示例一根据平台选择clean执行方式)
  - [6.3. 示例二：调试模式与发布模式](#63-示例二调试模式与发布模式)
- [7. 函数](#7-函数)
  - [7.1. `subst` —— 字符串替换](#71-subst--字符串替换)
  - [7.2. `patsubst` —— 模式替换](#72-patsubst--模式替换)
  - [7.3. `wildcard` —— 获取文件列表](#73-wildcard--获取文件列表)
  - [7.4. `notdir` —— 去掉路径，只保留文件名](#74-notdir--去掉路径只保留文件名)
  - [7.5. `dir` —— 获取路径部分](#75-dir--获取路径部分)
  - [7.6. `basename` 、 `addsuffix` 和 `addprefix` —— 批量处理文件名](#76-basename--addsuffix-和-addprefix--批量处理文件名)
  - [7.7. `shell` —— 执行 shell 命令](#77-shell--执行-shell-命令)
- [8. 完整示例程序](#8-完整示例程序)


## 1. 基础用法

一个典型的规则格式如下：

```make
target: dependencies
<TAB> command
```

- target：目标文件，比如可执行文件或中间文件。
- dependencies：依赖文件（源文件、头文件等）。
- command：生成目标所需要执行的命令（必须以 TAB 缩进 开头）。

示例：
```make
main.o: main.c
	gcc -c main.c -o main.o
```

!!! warning
    由于makfile对空格、tab极其敏感，建议编写时打开编辑器的空格、tab显示，并避免不必要的空格，规范化书写。


## 2. 执行逻辑[^1]

当我们执行 `make` 时，大致流程如下：

1. **解析 Makefile**
   - `make` 会从当前目录寻找 `Makefile` 或 `makefile` 文件。
   - 读取其中的规则、变量、伪目标等定义。

2. **确定默认目标**
   - 一般是文件中的第一个目标（例如 `app`）。
   - 也可以通过命令行指定，例如：  
```bash
make clean
```

3. **检查依赖关系**
   - 从目标开始，逐层检查依赖文件是否存在、是否比目标文件更新。
   - 如果依赖文件比目标文件“新”，说明目标需要重新生成。

4. **执行命令**
   - 对需要更新的目标，执行其规则中定义的命令。
   - 命令必须以 **TAB 缩进** 开头。

5. **递归构建**
   - 如果依赖文件本身也是其他规则的目标，则会递归检查和执行。
   - 直到所有依赖满足，才最终生成目标。

6. **结束**
   - 如果所有目标都已是最新，则 `make` 会提示：
```
make: 'app' is up to date.
```

### 2.1. 🔄 执行流程示意图

```text
          make
           │
           ▼
   读取并解析 Makefile
           │
           ▼
   确定要构建的目标 (默认/指定)
           │
           ▼
   检查目标的依赖文件
           │
    ┌──────┴────────┐
    │               │
依赖比目标旧      依赖比目标新/不存在
    │               │
目标已是最新    执行规则命令 → 生成新目标
```

!!! info "tips"
    可以使用 make -f [makefile_name] 指定使用某个makefile文件。
    
    
## 3. 伪目标 .PHONY

有些目标不是实际文件，而只是一个操作，例如 `clean`。  
这时建议使用 **.PHONY** 声明：

```make
.PHONY: clean  
clean:  
	rm -r  [filepath]
```
指定make目标文件：
```make
make clean
```

## 4. 变量

Makefile 支持变量，常用于保存编译器或编译选项。
使用示例如下：
```make
CC = gcc
CFLAGS = -Wall -g

app: main.o utils.o
	$(CC) $(CFLAGS) main.o utils.o -o app

main.o: main.c
	$(CC) $(CFLAGS) -c main.c -o main.o

utils.o: utils.c
	$(CC) $(CFLAGS) -c utils.c -o utils.o
```

### 4.1. 赋值符号

- **=** 
	我称之为最终赋值，同一个变量无论被赋值多次，永远取最后指定的值。
示例：
```make
VIR_A = A
VIR_B = $(VIR_A) B
VIR_A = AA
```
最后VIR_B的值是AA B。
- **:=**
	立即赋值，正常逻辑的赋值号，类似于c语言的赋值号。
- **?=**
	如果变量在之前没有被赋值则赋值。
	可以理解为
	```
	#ifndef
	#define ...
	#endif
	```
- **+=**
	追加赋值，将值追加到变量中。

## 5. 隐含规则与通配符

Make 内置了一些规则，可以用简写方式：
- `$@`：目标文件名
- `$<`：第一个依赖文件
- `$^`：所有依赖文件

`%` 表示可以匹配任意长度的字符串，用于定义一类文件的生成规则。例如：
```make
%.o: %.c     
	gcc -c $< -o $@
```
- 含义：
    - `%.o` 表示所有以 `.o` 结尾的目标文件。
    - `%.c` 表示所有以 `.c` 结尾的源文件。
    - `$<` 是第一个依赖文件（这里是 `.c` 文件）。
    - `$@` 是目标文件（这里是 `.o` 文件）。
- 作用：这条规则表示，所有 `.c` 文件可以通过编译生成对应的 `.o` 文件。

---

`%` 可以匹配文件名的某一部分，用于简化规则。例如：
```
build/%: src/%     
	cp $< $@
```
- 含义：
    - `build/%` 表示目标文件在 `build/` 目录下。
    - `src/%` 表示依赖文件在 `src/` 目录下。
    - `$<` 是依赖文件，`$@` 是目标文件。
- 作用：这条规则表示，将 `src/` 目录下的文件复制到 `build/` 目录下。

---

在模式规则中，`%` 可以用于定义多个目标。例如：
```
%.a: %.b %.c     
	cat $^ > $@`
```
- 含义：
    - `%.a` 是目标文件。
    - `%.b` 和 `%.c` 是依赖文件。
    - `$^` 表示所有依赖文件，`$@` 是目标文件。
- 作用：这条规则表示，将 `.b` 和 `.c` 文件合并生成 `.a` 文件。

---

## 6. 条件分支

在 Makefile 中，我们可以使用条件语句来根据不同情况执行不同规则或定义变量。常见的有 `ifeq`、`ifneq`、`ifdef`、`ifndef`。
### 6.1. 语法
```make
ifeq (条件1, 条件2)
    # 当 条件1 == 条件2 时执行这里
else
    # 否则执行这里
endif

ifneq (条件1, 条件2)
    # 当 条件1 != 条件2 时执行这里
endif

ifdef 变量名
    # 当变量已定义时执行这里
endif

ifndef 变量名
    # 当变量未定义时执行这里
endif
```

### 6.2. 示例一：根据平台选择clean执行方式
```make
# 默认变量
CC = gcc

# 判断系统
ifeq ($(OS), Windows_NT)
    RM = del
else
    RM = rm -f
endif

app: main.o
	$(CC) main.o -o app

clean:
	$(RM) *.o app
```

### 6.3. 示例二：调试模式与发布模式
```make
# 设置编译选项
CFLAGS = -Wall

ifeq ($(MODE), debug)
    CFLAGS += -g
else
    CFLAGS += -O2
endif

app: main.o
	$(CC) $(CFLAGS) main.o -o app
```
使用方式：
```bash
make MODE=debug   # 调试模式，带调试信息
make MODE=release # 默认优化模式
```

## 7. 函数

Makefile 内置了许多函数，用来处理字符串、文件名、路径等。  
常见函数格式为：

```make
$(函数名 参数1 参数2 ...)
```

下面介绍一些**常用函数**

### 7.1. `subst` —— 字符串替换

```make
$(subst from,to,text)
```

- 功能：将 `text` 中的 `from` 替换为 `to`。
    
- 示例：

```
SRC = main.c utils.c 
OBJ = $(subst .c,.o,$(SRC)) 

# 结果：OBJ = main.o utils.o
```

---

### 7.2. `patsubst` —— 模式替换

```make
$(patsubst pattern,replacement,text)
```

- 功能：更灵活的字符串替换，支持通配符 `%`。
    
- 示例：
```make
SRC = main.c utils.c test.c
OBJ = $(patsubst %.c,%.o,$(SRC))

# 结果：OBJ = main.o utils.o test.o
```

---

### 7.3. `wildcard` —— 获取文件列表

```make
$(wildcard pattern)
```

- 功能：匹配符合模式的文件。
    
- 示例：
```make
SRC = $(wildcard *.c)
# 结果：SRC = 当前目录下所有 .c 文件
```

---

### 7.4. `notdir` —— 去掉路径，只保留文件名

```make
FILES = src/main.c src/utils.c
NAMES = $(notdir $(FILES))
# 结果：NAMES = main.c utils.c
```
---

### 7.5. `dir` —— 获取路径部分

```make
FILES = src/main.c src/utils.c
PATHS = $(dir $(FILES))
# 结果：PATHS = src/ src/
```

---

### 7.6. `basename` 、 `addsuffix` 和 `addprefix` —— 批量处理文件名

```make
FILES = main.c utils.c

# 去掉后缀
NAMES = $(basename $(FILES))
# NAMES = main utils

# 批量添加后缀
OBJS = $(addsuffix .o,$(NAMES))
# OBJS = main.o utils.o

# 批量添加前缀
OBJS = $(addprefix -I,$(NAMES))
# OBJS = -Imain -Iutils
```

---

### 7.7. `shell` —— 执行 shell 命令

```make
DATE = $(shell date +%Y-%m-%d)
```

这样可以在 Makefile 中直接使用系统命令的输出。

## 8. 完整示例程序

这是一个完整Makefile示例程序，用于将c语言程序编译为可执行的二进制bin文件。它可以制成镜像供SoC烧录。

```make
#0###########################################################

# 设置目录变量，方便统一管理和修改
# 当前根目录:
ROOT_DIR := .

# 中间目标文件（.o）输出目录:
BUILD_DIR := build

# 最终生成的二进制文件（.bin）目录:
BIN_DIR := bin

# 工程名
NAME := key

# 指定链接脚本
LDS = imx.lds

#############################################################

#1###########################################################

# 自动查找 src/ 目录下的所有 .c 文件
SRCS = $(shell find $(ROOT_DIR) -name "*.c")

# 将 SRC中的 xxx.c 转换为 build/xxx.o
# 同时添加 build/startup.o（汇编启动文件）
OBJS = $(BUILD_DIR)/startup.o
OBJS += $(patsubst %.c,$(BUILD_DIR)/%.o,$(SRCS))

# 自动查找所有包含头文件的目录
INC_DIRS = $(shell find $(ROOT_DIR) -type f -name "*.h" -exec dirname {} \; | sort -u)
INCLUDES = $(addprefix -I, $(INC_DIRS))

#############################################################

#2###########################################################

# 设置编译工具（使用 ARM 的交叉编译工具链）
CC := arm-none-eabi-

# 编译器（用于 .c 和 .S 文件）:
GCC := $(CC)gcc

# 链接器:
LD := $(CC)ld

# 用于将 elf 转为 bin 格式:
OBJCOPY := $(CC)objcopy

# 用于反汇编
OBJDUMP := $(CC)objdump

# 编译选项（GCC 编译阶段）
# -I：指定头文件搜索目录
# -Wall：打开所有警告
# -O2：优化等级 2（推荐用于 release）
# -nostdlib：不链接标准库（适用于裸机）
# -c：只编译，不链接
GCC_FLAGS = $(INCLUDES) -Wall -nostdlib -c

# 链接器选项
LD_FLAGS = -T$(LDS)

# 使用 objdump 工具对生成的 ELF 文件进行反汇编
# -D：反汇编所有节（包括代码段、启动代码等）
# -m arm：指定目标架构为 ARM
# .elf：输入的可执行文件
# > .dis：将反汇编结果输出为 .dis 文本文件
OBJDUMP_FLAGS = -D -m arm $(BUILD_DIR)/$(NAME).elf > $(BUILD_DIR)/$(NAME).dis

#############################################################

#3###########################################################

# 目标：生成最终的二进制文件 bin/$(NAME).bin
$(BIN_DIR)/$(NAME).bin: $(OBJS)
	# 链接所有 .o 文件生成 elf 格式可执行文件
    $(LD) $(LD_FLAGS) $(OBJS) -o $(BUILD_DIR)/$(NAME).elf
	# 反汇编 调试用
    $(OBJDUMP) $(OBJDUMP_FLAGS)
	# 把 elf 文件转换为裸机二进制文件（无符号、无头信息）
    $(OBJCOPY) -O binary -S $(BUILD_DIR)/$(NAME).elf $@
 
# 编译汇编启动文件 startup.S，生成 build/startup.o
$(BUILD_DIR)/startup.o: startup.S
	# 注意 startup.S 是汇编文件，用 gcc 编译也可以，默认会调用汇编器
    $(GCC) $(GCC_FLAGS) $< -o $@

# 编译每个 .c 文件到 build/xxx.o
# $@：目标文件（例如 build/main.o）
# $<：依赖的源文件（例如 src/main.c）
$(BUILD_DIR)/%.o: %.c
	# 修复由于没有文件夹报错
    mkdir -p $(dir $@)
    $(GCC) $(GCC_FLAGS) -c $< -o $@

#############################################################

.PHONY: clean
clean:
    rm -r $(BUILD_DIR)/* $(BIN_DIR)/*
```



[^1]: 仅作了解

