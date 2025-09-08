---
title: Ubuntu下磁盘管理
tags:
  - Linux
  - Ubuntu
  - 磁盘管理
categories:
  - Linux
  - Ubuntu
abbrlink: 30265
date: 2025-06-12 16:00:30
---
<details>

<summary>版权信息</summary>

:::warning

本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。

:::

</details>

---

前面我们成功让WSL读取到Win系统的USB设备，下面来学习WSL下磁盘的管理。

## 磁盘和目录的容量查询命令

`df` ：查看文件系统使用量，主要查看文件系统的使用量。
`du` ：评估文件系统的磁盘使用量，主要查看单个文件大小。
`lsblk` ：*List Block Devices* 列出系统中SSD、U盘等存储设备的信息
## 磁盘的挂载卸载

还记得前面提到的[Ubuntu文件系统结构](https://blog.godmao.top/posts/7417/)吗？想要读取u盘里的文件，需要把u盘挂载在某个目录下才能访问。

### 什么是挂载？
挂载就是“**把设备上的文件系统接入Linux的目录结构中**”。
- Linux 的目录结构是一个**统一的树形结构**（根目录 `/` 是起点）。
- 所有的文件、文件夹、设备访问，都要通过这个统一的目录树。
- 当你插入一个U盘时，它本身是一个独立的文件系统（如FAT32），Linux不会自动将其连接进目录树，除非你明确地**挂载**它。

### 为什么Linux要挂载？Windows为什么不需要？
**Linux设计哲学：**
- Linux源于Unix，追求“一切皆文件”，统一的目录结构。
- 所有设备都必须通过`/dev/xxx`表示为一个文件，然后挂载到某个目录（如`/mnt/usb`）才可以访问。
**Windows的方式：**
- Windows是“盘符式”的，每个设备有独立的盘符（如`E:\`, `F:\`），操作系统会**自动识别并挂载**。
- 挂载过程对用户是透明的，但其实系统内部也在完成挂载，只是隐藏了细节。
所以：**Windows系统也挂载，只是自动化并且用盘符表示；Linux则更显式且灵活**。

### Linux下如何手动挂载u盘？

如果是U盘可以挂载在`/dev/media/`目录或`/dev/mnt/`目录
#### 1.查看u盘设备名
在使用usbipd使WSL读取到u盘后，运行如下命令：
```bash
lsblk
```

会看到类似这样的输出：
```
sdb      8:16   1  29.8G  0 disk
├─sdb1   8:17   1  10.0G  0 part
├─sdb2   8:18   1   9.8G  0 part
└─sdb3   8:19   1  10.0G  0 part
```
这里 `sdb1` 是U盘的分区名。磁盘可能有很多分区嘛。

#### 2.为每个分区创建挂载点目录
```bash
sudo mkdir -p /mnt/usb1 /mnt/usb2 /mnt/usb3
```

#### 3.逐个挂载
```bash
sudo mount /dev/sdb1 /mnt/usb1
sudo mount /dev/sdb2 /mnt/usb2
sudo mount /dev/sdb3 /mnt/usb3
```
如果某个分区的文件系统不是自动识别的（报错），你可以指定类型挂载，例如：
```bash
sudo mount -t vfat /dev/sdb1 /mnt/usb1
sudo mount -t ext4 /dev/sdb2 /mnt/usb2
sudo mount -t ntfs /dev/sdb3 /mnt/usb3
```
如果不清楚类型运行如下命令可以查看：
```bash
sudo blkid /dev/sdb1 /dev/sdb2 /dev/sdb3
```

### 卸载
```bash
sudo umount /mnt/usb1
sudo umount /mnt/usb2
sudo umount /mnt/usb3
```

## 磁盘分区

### fdisk命令常用选项
| 命令 | 含义                     | 功能说明                                     |
|------|--------------------------|----------------------------------------------|
| `m`  | help                     | 显示所有可用命令                             |
| `p`  | print                    | 显示当前分区表                               |
| `n`  | new                      | 创建一个新分区（主分区或逻辑分区）           |
| `d`  | delete                   | 删除一个已有分区                             |
| `t`  | type                     | 修改分区的类型 ID                            |
| `l`  | list                     | 列出所有分区类型 ID                         |
| `a`  | toggle bootable          | 切换分区的启动标志（设置/取消可引导分区）   |
| `w`  | write                    | 写入分区表并退出                             |
| `q`  | quit                     | 不保存更改，直接退出                         |
| `g`  | gpt                      | 创建新的 GPT 分区表                          |
| `o`  | dos                      | 创建新的空白 DOS (MBR) 分区表                |
| `v`  | verify                   | 验证分区表的完整性                           |
| `x`  | expert                   | 进入专家模式（用于高级操作）                 |
### 用fdisk打开要分区的磁盘
```bash
sudo fdisk /dev/sde
```

:::warning
千万不要误选，否则可能破坏你的数据。
:::

### 删除分区
```bash
Command (m for help): p        ← 查看当前分区情况（确认要删哪个）
Command (m for help): d        ← 删除分区
Partition number (1,2,...): 2  ← 选择你要删除的分区编号（比如 2 表示 /dev/sdb2）
Command (m for help): w        ← 保存更改并退出
```

### 创建分区
```bash
Command (m for help): n
```

![](Snipaste_2025-06-12_17-21-39.png)

1：分区编号，直接回车接受默认编号。
2：起始扇区，默认值通常合适，直接回车即可。
3：结束扇区 or 分区大小，可以输入：扇区编号（直接回车使用剩余空间）或者手动输入大小（例如：`+1G`、`+512M`）

### 写出
保存并退出。
```bash
Command (m for help): w
```

## 格式化分区
当用 `fdisk` 创建完一个分区后，这个分区只是“逻辑划分好了”，但**还不能使用**，必须要**格式化（创建文件系统）**才能真正读写数据。

### 为什么要格式化？
格式化的本质是：

> **在分区上写入特定的文件系统结构**（如 ext4、FAT32、NTFS 等）。

只有这样，操作系统才知道如何在这块区域内组织、存储和查找文件。

### 1.选择格式化类型
| 文件系统  | 说明                               |
| ----- | -------------------------------- |
| ext4  | Linux常用，性能稳定，支持大文件               |
| vfat  | FAT32，兼容Windows/Mac/Linux，不支持大文件 |
| ntfs  | Windows常用，Linux可读写（需驱动）          |
| exfat | 新型通用格式，支持大文件，跨平台兼容性好             |
### 2.执行格式化命令
*make file system* -type
```bash
sudo mkfs.vfat -f 32 /dev/sde2
```
没有mkfs的先安装，不然报错：
```
failed to execute mkfs.vfat: No such file or directory
```
安装：
```bash
sudo apt update
sudo apt install dosfstools
```

### 检查
可以输入命令：
```bash
sudo blkid /dev/sde1 /dev/sde2
```
检查type
输出：
```
sde1: UUID="FBD5-E06C" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="3b15f93b-f163-491d-9f0f-94be96692beb"
sde2: UUID="FC22-7890" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="248a2fdb-2255-4a7a-802e-e5a11bb87c53"
```
