---
title: mx6ull开发版移植nxp官方u-boot
layout: post
tags:
  - u-boot移植
  - Linux
categories:
  - Linux
  - 系统移植
abbrlink: 26288
date: 2025-10-29 16:25:32
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

## 1. 我写尼玛！

我他妈直接不想码字。
我他妈直接转载。

移植部分：
作者：觉皇嵌入式 遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议。
[i.MX6ULL - 从零开始移植uboot-imx_v2020.04_5.4.70_2.3.0-CSDN博客](https://blog.csdn.net/qq153471503/article/details/126587387)

## 2. 图形化配置

在uboot目录下，使用命令：

```
make menuconfig
```

可以通过图形化界面配置uboot，更加直观。

## 3. U-boot 启动Linux测试

### 3.1. 从EMMC启动

#### 3.1.1. 查看

首先检查EMMC里面是否有——Linux镜像zImage、.dtb文件和根文件系统。

我们当前的mmc环境是SD卡，切换到EMMC。输入命令：

```
mmc dev 1
```

>具体EMMC的设备号 根据实际情况输入。

该开发板EMMC分区是这样的：
- 分区0：是存放boot的，
- 分区1：存放linux镜像和设备树，
- 分区2：存放root。

>可以使用 `fstype mmc dev:part` 命令查看文件系统格式。
>其中 dev是EMMC的设备号，part是分区号。运行后：
>分区0输出 `Unrecognized...`；
>分区1输出 `fat` ；分区2输出 `ext4`。

分区1使用的文件系统是FAT，因此用 `fatls` 查看EMMC分区1里面的文件：

```
fatls mmc dev:part
```

输出：

```
7863120   zImage
    38733   imx6ull-mmc-npi-lite.dtb

2 file(s), 0 dir(s)
```

说明系统和设备树是存在的。

#### 3.1.2. 加载系统相关命令

- **查看指定的镜像加载地址：**

	`printenv loadaddr`

	输出为 `0x80800000`

- **查看指定的设备树加载地址：**

	`printenv fdt_addr`

	输出为 `0x83000000`

- **把zImage加载到地址 `0x80800000`：**

	`fatload mmc 1:1 80800000 zImage`

- **把设备树加载到地址 `0x83000000`**：

	`fatload mmc 1:1 83000000 imx6ull-mmc-npi-lite.dtb`

- **启动内核：**
	`bootz 80800000 - 83000000`

#### 3.1.3. 设置bootcmd

`bootcmd` 是一个环境变量，保存着 uboot 默认命令，uboot 倒计时结束以后就会执行 bootcmd 中的命令。这些命令一般都是用来启动 Linux 内核的，比如读取 EMMC 或者 NAND Flash 中的 Linux 内核镜像文件和设备树文件到 DRAM 中，然后启动 Linux 内核。

```
setenv bootcmd 'mmc dev 1; fatload mmc 1:1 80800000 zImage; fatload mmc 1:1 83000000 imx6ull-mmc-npi-lite.dtb; bootz 80800000 - 83000000;'
```

#### 3.1.4. 设置bootargs

<details>

<summary> 关于bootargs的介绍有点长，折叠一下。</summary>

bootargs 保存着 uboot 传递给 Linux 内核的参数，bootargs 环境变量是由 mmcargs 设置的，mmcargs 环境变量如下：
`mmcargs=setenv bootargs console=${console},${baudrate} root=${mmcroot}`

其中 console=ttymxc0，baudrate=115200，mmcroot=/dev/mmcblk0p2 rootwait rw，

因此将mmcargs 展开以后就是：
`mmcargs=setenv bootargs console= ttymxc0, 115200 root= /dev/mmcblk0p2 rootwait rw`

可以看出环境变量 mmcargs 就是设置 bootargs 的值为`console= ttymxc0, 115200 root=/dev/mmcblk1p2 rootwait rw`，

bootargs 就是设置了很多的参数的值，这些参数 Linux 内核会使用到，常用的参数有：

- **console：**
	console 用来设置 linux 终端(或者叫控制台)，也就是通过什么设备来和 Linux 进行交互，例如串口、 LCD 屏幕。如果是串口的话应该是串口几、波特率等。一般设置串口作为 Linux 终端，这样我们就可以在电脑上通过串口程序来和 linux 交互了。这里设置 console 为 ttymxc0，因为 linux启动以后 I.MX6ULL 的串口 1 在 linux 下的设备文件就是/dev/ttymxc0，”一切皆文件“。
- **root：**
	root 用来设置根文件系统的位置，例如，`root=/dev/mmcblk1p2` 指根文件系统存放在 mmcblk1 设备的分区 2 中。
	`rootwait rw`。rootwait 表示等待 mmc 设备初始化完成以后再挂载，否则的话mmc 设备还没初始化完成就挂载根文件系统会出错的。rw 表示根文件系统是可以读写的，不加 rw 的话可能无法在根文件系统中进行写操作，只能进行读操作。
- **rootfstype：**
	此选项一般配置 root 一起使用，rootfstype 用于指定根文件系统类型，如果根文件系统为ext 格式的话此选项无所谓。如果根文件系统是 yaffs、jffs 或 ubifs 的话就需要设置此选项，指定根文件系统的类型。

</details>

设置：
```
setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw'
```

#### 3.1.5. 启动

使用命令 `run bootcmd` 或 在切换到EMMC环境后直接输入`boot`。

### 3.2. 从网络启动

#### 3.2.1. 安装 tftpd-hpa 和客户端工具

```bash
sudo apt update
sudo apt install tftpd-hpa tftp-hpa
```

#### 3.2.2. 创建共享目录并开放权限

TFTP 的默认根目录是 `/srv/tftp`，你也可以改成别的：

```bash
sudo mkdir -p /srv/tftp
sudo chmod 777 /srv/tftp
```

然后把要传的文件（比如 `uImage`）放进去：

```bash
cp ~/uImage /srv/tftp/
```

#### 3.2.3. 手动启动 TFTP 服务（因为没有 systemctl）

直接运行：

```bash
sudo /usr/sbin/in.tftpd --foreground --secure --verbose /srv/tftp
```

- --foreground：在前台运行
- --secure：只允许访问指定目录
- --verbose：打印详细日志

> 还有一些选项，例如：
> --address <ip:port>：指定 TFTP 服务监听的 IP 地址和端口。
> --blocksize <大小>：指定一个数据包的大小。
> --create：允许客户端上传。

这会在前台启动服务器（按 `Ctrl+C` 可停止）。

如果你想让它**后台运行**：

```bash
sudo /usr/sbin/in.tftpd --secure /srv/tftp &
```

#### 3.2.4. 确认服务在监听 UDP 69 端口

```bash
sudo netstat -anu | grep :69
```

或者：

```bash
sudo ss -anu | grep 69
```

出现类似：

```nginx
udp   UNCONN  0      0      0.0.0.0:69    0.0.0.0:*
```

说明启动成功。

#### 3.2.5. 让 U-Boot 能访问 WSL 的 IP

这是关键部分。
WSL 和 Windows 的网络是**桥接但不完全共享的**，所以开发板不能直接访问 `127.0.0.1` 或 `localhost`。

1. 在 Windows 里执行：

```powershell
ipconfig
```
找到 **以太网适配器 vEthernet (WSL)** 这一项。
它通常是类似 `172.25.x.x` 或 `192.168.137.x` 的 IP，比如：

```nginx
IPv4 Address. . . . . . . . . . . : 172.27.240.1
```
2. 在 U-Boot 里设置：

```bash
setenv serverip 192.168.137.1
setenv ipaddr 192.168.137.144
```

3. 关闭防火墙：
	在设置——网络与Internet——高级网络设置——Windows防火墙，将公用网络中的防火墙暂时关闭。
	
4. 测试：

```bash
ping 192.168.137.1
```
如果显示 “host is alive”，说明连通。

#### 3.2.6. 传文件

假设你要下载 `/srv/tftp/uImage` 到内存地址 `0x80800000`：

```bash
tftp 0x80800000 uImage
```

看到类似输出：

```nginx
Using FEC device
TFTP from server 172.27.240.1; our IP address is 172.27.240.166
Filename 'uImage'.
Load address: 0x80800000
Loading: #########################################
done
Bytes transferred = 3456789 (34abcd hex)
```

就成功了！
