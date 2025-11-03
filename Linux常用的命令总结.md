---
title: Linux常用的命令总结
tags:
  - Linux
  - 命令
categories:
  - Linux
  - 入门
abbrlink: 37616
date: 2025-06-11 15:35:35
---
<details>

<summary>版权信息</summary>

:::warning

本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。

:::

</details>

---

Linux下常用的命令清单及补充。
<!--more-->

**仅作学习记录。**

| 命令         | 说明                 | 使用格式                   |
| ---------- | ------------------ | ---------------------- |
| `ls`       | 列出目录内容             | `ls [选项] [目录]`         |
| `cd`       | 改变当前目录             | `cd [目录]`              |
| `pwd`      | 显示当前工作目录           | `pwd`                  |
| `cp`       | 复制文件或目录            | `cp [选项] 源文件 目标文件`     |
| `mv`       | 移动文件或目录，重命名文件      | `mv [选项] 源文件 目标文件`     |
| `rm`       | 删除文件或目录            | `rm [选项] 文件`           |
| `touch`    | 创建空文件              | `touch 文件名`            |
| `mkdir`    | 创建目录               | `mkdir [选项] 目录名`       |
| `rmdir`    | 删除空目录              | `rmdir 目录名`            |
| `chmod`    | 更改文件或目录权限          | `chmod [选项] 权限 文件`     |
| `chown`    | 更改文件或目录的所有者        | `chown 用户:组 文件`        |
| `chgrp`    | 更改文件或目录的所属组        | `chgrp 组 文件`           |
| `cat`      | 查看文件内容             | `cat 文件名`              |
| `more`     | 分页显示文件内容           | `more 文件名`             |
| `less`     | 分页显示文件内容（支持上下翻页）   | `less 文件名`             |
| `head`     | 查看文件开头部分           | `head [选项] 文件`         |
| `tail`     | 查看文件尾部部分           | `tail [选项] 文件`         |
| `find`     | 查找文件或目录            | `find [路径] [选项] [表达式]` |
| `grep`     | 文本搜索               | `grep [选项] "模式" 文件`    |
| `tar`      | 压缩或解压文件            | `tar [选项] [文件]`        |
| `gzip`     | 压缩文件               | `gzip 文件名`             |
| `gunzip`   | 解压.gz文件            | `gunzip 文件名.gz`        |
| `zip`      | 压缩文件               | `zip [选项] 压缩包 文件`      |
| `unzip`    | 解压.zip文件           | `unzip 压缩包`            |
| `df`       | 查看文件系统磁盘空间使用情况     | `df [选项]`              |
| `du`       | 查看目录或文件的磁盘使用情况     | `du [选项] 文件/目录`        |
| `top`      | 查看系统进程状态           | `top`                  |
| `ps`       | 查看进程状态             | `ps [选项]`              |
| `kill`     | 终止进程               | `kill [选项] 进程号`        |
| `pstree`   | 以树形结构显示进程          | `pstree`               |
| `free`     | 查看内存使用情况           | `free [选项]`            |
| `uname`    | 查看系统信息             | `uname [选项]`           |
| `ifconfig` | 配置网络接口             | `ifconfig [网络接口]`      |
| `ip`       | 查看或配置网络            | `ip [选项]`              |
| `ping`     | 测试网络连接             | `ping [选项] 地址`         |
| `scp`      | 安全复制文件             | `scp 源文件 用户@主机:目标`     |
| `rsync`    | 同步文件和目录            | `rsync [选项] 源 目标`      |
| `wget`     | 从网络下载文件            | `wget [选项] URL`        |
| `curl`     | 与网络交互（下载、上传文件等）    | `curl [选项] URL`        |
| `alias`    | 创建命令别名             | `alias 别名='命令'`        |
| `history`  | 查看命令历史             | `history`              |
| `man`      | 查看命令手册             | `man 命令名`              |
| `echo`     | 输出字符串到终端           | `echo "文本"`            |
| `tee`      | 从标准输入读取，并将其内容输出到文件 | `命令 \| tee 文件`         |
| `cut`      | 按列切割文件内容           | `cut -d 分隔符 -f 列 文件`   |
| `awk`      | 强大的文本处理工具          | `awk '条件 {动作}' 文件`     |
| `sed`      | 流编辑器，处理文本数据        | `sed 's/模式/替换文本/' 文件`  |
| `tr`       | 转换字符               | `tr '旧字符' '新字符' < 文件`  |
| `wc`       | 统计文件字数、行数、字节数等     | `wc [选项] 文件`           |
| `whoami`   | 查看当前用户             | `whoami`               |
| `sudo`     | 以超级用户身份执行命令        | `sudo 命令`              |
| `exit`     | 退出终端或当前Shell会话     | `exit`                 |
| `file`     | 文件类型查看命令           |  `file 文件路径`           |

## ls
-a 列出全部
-l 以列展示

## ranger
一个终端文件管理器（Vim风格）

| 操作        | 快捷键                |
| --------- | ------------------ |
| 向上/向下     | 方向键                |
| 进入目录或打开文件 | `l / Enter`        |
| 返回上一级目录   | 方向键                |
| 创建新文件     | `:touch filename`  |
| 创建新目录     | `:mkdir dirname`   |
| 删除文件/目录   | `D` 然后确认           |
| 重命名       | `cw`               |
| 复制        | `yy`（复制）+ `pp`（粘贴） |
| 剪切        | `dd`（剪切）+ `pp`（粘贴） |
| 预览文件内容    | 自动或按 `i`           |
| 搜索文件名     | `/关键词`             |
| 退出        | `q` 或 `:q`         |

## grep

`grep` 是 Linux 系统中一个强大的文本搜索工具，用于在文件中查找符合条件的字符串或正则表达式，并打印匹配的行。它的名字来源于 "Global Regular Expression Print"。
### 基本语法

`grep [选项] "搜索内容" 文件名`

### 常用选项

1. **`-i`**：忽略大小写。

    `grep -i "hello" file.txt`
    - 匹配 `hello`、`HELLO` 等。
    
2. **`-n`**：显示匹配行的行号。

    `grep -n "hello" file.txt`
    - 输出匹配行及其行号。
    
3. **`-v`**：反向匹配，显示不包含搜索内容的行。
    
    `grep -v "hello" file.txt`
    
4. **`-c`**：仅显示匹配的行数。
    
    `grep -c "hello" file.txt`
    
5. **`-r` 或 `-R`**：递归搜索目录中的文件。
    
    `grep -r "hello" /path/to/directory`
    
6. **`-l`**：仅显示包含匹配内容的文件名。
    
    `grep -l "hello" *.txt`
    
7. **`-w`**：匹配整个单词。
    
    `grep -w "hello" file.txt`
    
8. **`-A` 和 `-B`**：显示匹配行的上下文。
    
    - **`-A`**：显示匹配行后面的若干行。
        `grep -A 2 "hello" file.txt`
        
    - **`-B`**：显示匹配行前面的若干行。
        `grep -B 2 "hello" file.txt`
        
    - **`-C`**：同时显示匹配行的前后若干行。
        `grep -C 2 "hello" file.txt`
        

### 正则表达式支持

`grep` 支持基本正则表达式（BRE）和扩展正则表达式（ERE，需使用 `egrep` 或 `grep -E`）。

- **匹配任意字符**：`grep "h.llo" file.txt` 匹配 `hello`、`hallo` 等。
- **匹配开头**：`grep "^hello" file.txt` 匹配以 `hello` 开头的行。
- **匹配结尾**：`grep "hello$" file.txt` 匹配以 `hello` 结尾的行。
- **匹配多个选项**：`grep -E "hello|world" file.txt` 匹配 `hello` 或 `world`。

### 实例

1. 查找文件中包含 "error" 的行并显示行号：

    `grep -n "error" log.txt`
    
2. 在当前目录及子目录中查找包含 "TODO" 的文件：

    `grep -r "TODO" .`
    
3. 查找以 "INFO" 开头的日志行：

    `grep "^INFO" log.txt`
    
4. 查找不包含 "DEBUG" 的行：

    `grep -v "DEBUG" log.txt`

## du
-s 只显示指定目录总大小 
-h 以单位k、m、g显示

## tar

`tar [选项] -f 归档文件名 [文件或目录]`
### 选项：
- -c：创建新的归档文件。
- -x：解压归档文件。
- -t：列出归档文件的内容。
- -v：显示详细操作过程。
- -z：使用 gzip 压缩或解压。
- -j：使用 bzip2 压缩或解压。
- -J：使用 xz 压缩或解压。
- --delete：从归档文件中删除指定文件（仅限 GNU tar）。

### 常见操作示例：

1. **创建归档文件**

	将文件 _file1_ 和 _file2_ 以及目录 _dir_ 打包成 _archive.tar_：
	`tar -cvf archive.tar file1 file2 dir`

2. **解压归档文件**

	解压 _archive.tar_ 到当前目录：
	`tar -xvf archive.tar`

3. **压缩归档文件**

	将目录 _dir_ 打包并使用 gzip 压缩为 _archive.tar.gz_：
	`tar -czvf archive.tar.gz dir`

4. **解压压缩归档文件**

	解压 _archive.tar.gz_：
	`tar -zxvf archive.tar.gz`

5. **列出归档文件内容**

	查看 _archive.tar_ 中的文件和目录：
	`tar -tvf archive.tar`

6. **向归档文件追加文件**

	将 _newfile_ 添加到已存在的 _archive.tar_ 中：
	`tar -rvf archive.tar newfile`

7. **删除归档文件中的文件**

	从 _archive.tar_ 中删除 _file1_：
	`tar --delete -f archive.tar file1`

8. **解压到指定目录**

	将 _archive.tar.gz_ 解压到 _/tmp/files_ 目录：
	`tar -zxvf archive.tar.gz -C /tmp/files`

### **注意事项**

- 使用 _-v_ 可以查看详细的操作过程。 
- 解压时，_-C_ 选项可以指定目标目录。
- 压缩时，建议根据文件类型选择合适的压缩算法（如 gzip、bzip2 或 xz）。

## dd

### dd 命令简介

`dd` 是 Linux 和类 UNIX 系统中一个功能强大的命令行工具，主要用于低级别的数据复制和转换操作。它可以处理设备文件、磁盘、分区、文件等，常用于备份、恢复、创建磁盘镜像等任务。

`dd if=<输入文件> of=<输出文件> [参数...]`

- **if**：指定输入文件（input file），如 `/dev/zero`、磁盘设备或普通文件。
- **of**：指定输出文件（output file），如磁盘设备或普通文件。
- **参数**：用于控制块大小、数据量等。
### 常用参数

- **bs=大小**：设置块大小（如 `bs=1M` 表示每次读写 1MB 数据）。
- **count=数量**：指定复制的块数。
- **skip=数量**：跳过输入文件的前若干块。
- **seek=数量**：跳过输出文件的前若干块。
- **status=progress**：显示进度信息。

### 常见用法

1. **创建空文件**
    
    `dd if=/dev/zero of=empty_file bs=1M count=10`
    
    - 创建一个大小为 10MB 的空文件。
2. **制作磁盘镜像**
        
    `dd if=/dev/sda of=/path/to/image.img bs=1M`
    
    - 将整个磁盘 `/dev/sda` 制作成镜像文件。
3. **恢复磁盘镜像**
    
    `dd if=/path/to/image.img of=/dev/sda bs=1M`
    
    - 将镜像文件写回磁盘。
4. **清空磁盘数据**
        
    `dd if=/dev/zero of=/dev/sda bs=1M`
    
    - 用零填充整个磁盘 `/dev/sda`，清空数据。
5. **测试磁盘读写速度**
    
    `dd if=/dev/zero of=testfile bs=1G count=1 oflag=direct`
    
    - 创建一个 1GB 的文件，测试写入速度。

### 注意事项

- `dd` 是一个强大的工具，但操作不当可能导致数据丢失，尤其是涉及磁盘设备时，请务必确认输入和输出路径。
- 使用 `status=progress` 参数可以实时查看操作进度。

## find

`find` 是 Linux 中非常强大的命令，用于在指定目录下递归查找文件或目录，并支持多种条件过滤和操作。以下是它的基本用法和常见示例：

### 基本语法

`find [路径] [选项] [表达式]`

- **路径**：指定查找的目录（如 `/`, `.` 表示当前目录）。
- **选项**：控制查找行为（如按文件名、大小、时间等）。
- **表达式**：定义匹配条件或操作（如执行命令）。

### 常用选项

1. **按文件名查找**

    `find /path -name "filename"`
    - 区分大小写：`-name`
    - 不区分大小写：`-iname`

2. **按文件类型查找**

    `find /path -type [f|d|l]`

    - `f`：普通文件
    - `d`：目录
    - `l`：符号链接

3. **按文件大小查找**

    `find /path -size [+|-]N[c|k|M|G]`
    
    - `+N`：大于 N 单位
    - `-N`：小于 N 单位
    - `N`：等于 N 单位
    - 单位：`c`（字节）、`k`（KB）、`M`（MB）、`G`（GB）

4. **按修改时间查找**

    `find /path -mtime [+|-]N`
    
    - `+N`：N 天前修改
    - `-N`：N 天内修改
    - `N`：正好 N 天前修改
    - 访问时间：`-atime`，状态更改时间：`-ctime`

5. **按权限查找**

    `find /path -perm [mode]`
    
    - 精确匹配：`-perm 644`
    - 至少包含权限：`-perm -644`

6. **按用户或组查找**

    `find /path -user username find /path -group groupname`

### 执行操作

1. **删除匹配的文件**
    
    `find /path -name "*.log" -delete`
    
    或

    `find /path -name "*.log" -exec rm -f {} \;`
    
2. **查找并执行命令**
    
    `find /path -type f -name "*.txt" -exec cat {} \;`
    
    - `{}`：表示当前匹配的文件
    - `\;`：表示命令结束

3. **查找并移动文件**
    
    `find /path -name "*.jpg" -exec mv {} /new/path/ \;`
    

### 综合示例

1. 查找当前目录下所有 `.txt` 文件：

    `find . -name "*.txt"`
    
2. 查找 `/var/log` 目录下最近 7 天修改的文件：

    `find /var/log -mtime -7`
    
3. 查找 `/home` 目录下大于 100MB 的文件：
    
    `find /home -size +100M`
    
4. 查找 `/tmp` 目录下权限为 777 的文件并删除：

    `find /tmp -perm 777 -delete`
    