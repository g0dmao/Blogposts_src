---
title: Shell脚本流程控制简明指南
tags:
  - Linux
  - shell
categories:
  - Linux
  - Shell
abbrlink: 21870
date: 2025-06-18 18:38:30
---
<details>

<summary>版权信息</summary>

:::warning

本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。

:::

</details>

---

- [1. 条件判断语句 if/else](#1-条件判断语句-ifelse)
  - [1.1. 基本语法：](#11-基本语法)
  - [1.2. 示例：](#12-示例)
- [2. 判断符号说明](#2-判断符号说明)
- [3. case 分支结构](#3-case-分支结构)
  - [3.1. 语法：](#31-语法)
  - [3.2. 示例：](#32-示例)
- [4. for 循环](#4-for-循环)
  - [4.1. 遍历列表：](#41-遍历列表)
  - [4.2. 使用 C 风格语法：](#42-使用-c-风格语法)
- [5. while 循环](#5-while-循环)
  - [5.1. 基本语法：](#51-基本语法)
  - [5.2. 示例：](#52-示例)
- [6. until 循环](#6-until-循环)
- [7. 跳出循环：break 和 continue](#7-跳出循环break-和-continue)


Shell 脚本不仅可以批量处理命令任务，还拥有完整的流程控制语法结构，包括条件判断、循环与分支等逻辑控制结构。本文将简明介绍 Shell 的基本流程控制语法，适用于 bash 环境。

## 1. 条件判断语句 if/else

### 1.1. 基本语法：

```shell
if [ 条件 ]; then
    命令1
elif [ 条件 ]; then
    命令2
else
    命令3
fi
```
写成一行：
```shell
if [ ]; then ; fi
```

### 1.2. 示例：

```shell
read -p "请输入一个数字: " num
if [ "$num" -gt 0 ]; then
    echo "正数"
elif [ "$num" -lt 0 ]; then
    echo "负数"
else
    echo "零"
fi
```

---

## 2. 判断符号说明

|条件表达式|含义|
|---|---|
|`-eq`|等于（整数）|
|`-ne`|不等于|
|`-gt`|大于|
|`-lt`|小于|
|`-ge`|大于等于|
|`-le`|小于等于|
|`-z str`|字符串是否为空|
|`-n str`|字符串是否非空|
|`str1 = str2`|字符串相等|
|`-f file`|是否为普通文件|
|`-d dir`|是否为目录|

> 注意：`[` 和 `]` 要有空格；变量最好加引号防止空值。

## 3. case 分支结构

用于多个条件的匹配处理。

### 3.1. 语法：

```shell
case $变量 in
    模式1)
        命令 ;;
    模式2)
        命令 ;;
    *)
        默认命令 ;;
esac
```

### 3.2. 示例：

```shell
read -p "请输入选项[a/b/c]: " choice
case $choice in
    a) echo "你选择了A" ;;
    b) echo "你选择了B" ;;
    c) echo "你选择了C" ;;
    *) echo "无效选项" ;;
esac
```

## 4. for 循环

### 4.1. 遍历列表：

```shell
for var in 值1 值2 值3; do
    命令
done
```

```shell
for name in Alice Bob Charlie; do
    echo "Hello, $name"
done
```
### 4.2. 使用 C 风格语法：

```bash
for ((i=1; i<=5; i++)); do
    echo "第 $i 次"
done
```

## 5. while 循环

### 5.1. 基本语法：

```shell
while [ 条件 ]; do
    命令
done
```
### 5.2. 示例：

```shell
i=1
while [ $i -le 5 ]; do
    echo "循环第 $i 次"
    ((i++))
done
```

## 6. until 循环

与 `while` 相反：**条件为假时执行**

```shell
i=1
until [ $i -gt 5 ]; do
    echo "第 $i 次"
    ((i++))
done
```

## 7. 跳出循环：break 和 continue

```shell
for ((i=1;i<=10;i++)); do
    if [ $i -eq 5 ]; then
        break   # 提前结束循环
    fi
    echo $i
done
```

```shell
for ((i=1;i<=5;i++)); do
    if [ $i -eq 3 ]; then
        continue   # 跳过当前循环
    fi
    echo $i
done
```