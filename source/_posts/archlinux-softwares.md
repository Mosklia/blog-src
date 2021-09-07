---
title: ArchLinux 软件记录
date: 2021-09-07 19:40:14
tags:
- Linux
- ArchLinux
- 软件
categories:
- 调教电脑
---

这里主要记录一些玩 ArchLinux 时，推荐用的一些软件，或者是一些常用软件遇到问题的解决方案。
<!-- more -->

# KDE 相关软件
## KDE Connect

我个人最喜欢的一个软件，只要连接到同一局域网（似乎包括手机开热点），真正意义上的手机电脑同步协作（除了不能投屏）。

### 无法访问手机文件系统

这个功能是由软件包 `sshfs` 提供的，如果没有安装，请安装：

```sh
sudo pacman -S sshfs
```

## Okular

### 不显示 PDF 等文件中的中文

```sh
sudo pacman -S poppler-data
```

## Ark

### 无法打开 rar 压缩文件

```sh
sudo pacman -S unrar
```

## Discover

### 无法确定发行版的后端

```sh
sudo pacman -S archlinux-appstream-data packagekit-qt5 flatpak fwupd
```

# 腾讯系软件

## QQ

QQ 似乎总是会出点问题。建议安装 Tim：

```sh
pikaur -S com.qq.tim.spark
```