---
title: WSL-Archlinux安装后应该做什么？
layout: post
tags:
  - WSL2
  - ArchLinux
categories:
  - Linux
  - ArchLinux
abbrlink: 17327
date: 2025-09-18 20:21:15
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

- [1. 在WSL上安装Archlinux](#1-在wsl上安装archlinux)
- [2. 更新系统](#2-更新系统)
- [3. 基础配置](#3-基础配置)
  - [3.1. 设置root密码](#31-设置root密码)
  - [3.2. 创建普通用户并加入 `wheel` 组（方便后续 sudo 权限）：](#32-创建普通用户并加入-wheel-组方便后续-sudo-权限)
  - [3.3. 编辑 sudo 权限](#33-编辑-sudo-权限)
  - [3.4. 设定默认用户](#34-设定默认用户)
  - [3.5. 设置语言和地区](#35-设置语言和地区)
  - [3.6. AUR 支持](#36-aur-支持)
  - [3.7. 关于换源](#37-关于换源)
- [4. 个性化设置](#4-个性化设置)
  - [4.1. 使用zsh替代bash](#41-使用zsh替代bash)
  - [4.2. 使用oh-my-zsh为zsh提供插件服务](#42-使用oh-my-zsh为zsh提供插件服务)
  - [4.3. 使用oh-my-posh为zsh提供主题](#43-使用oh-my-posh为zsh提供主题)


## 1. 在WSL上安装Archlinux

请参见—— [在 WSL 上安装 Arch Linux - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/%E5%9C%A8_WSL_%E4%B8%8A%E5%AE%89%E8%A3%85_Arch_Linux)

## 2. 更新系统

```bash
sudo pacman -Syu
```
## 3. 基础配置

### 3.1. 设置root密码
```bash
passwd
```

### 3.2. 创建普通用户并加入 `wheel` 组（方便后续 sudo 权限）：
```bash
useradd -m -G wheel -s /bin/bash yourname
passwd yourname
```

`-m`：自动创建用户目录
`-G`：指定用户组
`-s`：指定shell

### 3.3. 编辑 sudo 权限
安装 `sudo`，再启用 `wheel` 组：
```bash
pacman -S sudo nano
```

```bash
nano /etc/sudoers
```

去掉 `# %wheel ALL=(ALL:ALL) ALL` 前的注释

!!! todo "tip"
    前面带%(如%wheel)则表示这是一个用户组，不带则表示这是一个用户。


### 3.4. 设定默认用户
不能总是用root用户登录。
首先确保该用户已被创建，然后将以下行添加到 `/etc/wsl.conf`：

```
[user]
default=username
```

### 3.5. 设置语言和地区
目的是正确显示中文字符
```bash
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

请参见——[简体中文本地化 - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E6%9C%AC%E5%9C%B0%E5%8C%96)

### 3.6. AUR 支持
可以下载来自社区的软件包，这里选用AUR助手 yay
选择一个指定目录然后执行：
```bash
pacman -S git base-devel --needed  #<-这是基本的开发工具包，如gcc、make
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si # 构建包
```
`-s`：自动安装依赖
`-i`：编译完成后自动安装生成的包

请参见——[yay - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/Yay)

### 3.7. 关于换源

打开`WSL-setting`（直接在win11里搜索）的网络设置，将网络模式由`NAT` 改为 `mirror`， 则可直接继承宿主机的网络环境。在宿主机使用网络安全工具即可，因此暂时没有换源。

## 4. 个性化设置

### 4.1. 使用zsh替代bash

```bash
sudo pacman -S zsh
chsh -s /bin/zsh # 改变默认shell
```

### 4.2. 使用oh-my-zsh为zsh提供插件服务
```bash
yay -S oh-my-zsh.git
```

安装完成后 可以在 `/usr/share/oh-my-zsh` 中查看主题文件和插件以及其提供的`zshrc`配置模板，直接使用配置模板。
```bash
cp /usr/share/oh-my-zsh/zshrc ~/.zshrc
```

进入`/usr/share/oh-my-zsh\plugins`目录 安装自动补全与语法高亮插件（不自带）
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions
```

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
```

在 `.zshrc` 里添加：
```
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

### 4.3. 使用oh-my-posh为zsh提供主题
你也可以使用`oh-my-zsh`的主题，只不过我更喜欢`oh-my-posh`的主题

```bash
yay -S oh-my-posh-bin
```

同样，安装完成后 可以在 `/usr/share/oh-my-posh` 中查看主题文件，
编辑`~/.zshrc`：
```bash
#注释掉ZSH_THEME="robbyrussell"
eval "$(oh-my-posh init zsh --config 'amro')"
```

!!! error "bug"
    建议不要使用类似于 `--config ~/.poshthemes/mytheme.omp.json`的选项，这可能会出现问题。


重新加载：
```bash
source ~/.zshrc
```

