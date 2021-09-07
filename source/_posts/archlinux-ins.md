---
title: ArchLinux/Windows 双系统安装记录
date: 2021-09-06 15:01:37
tags: 
- Linux
- ArchLinux
- 双系统
categories:
- 调教电脑
---

离我第一次入坑 [ArchLinux](https://wiki.archlinux.org/title/Main_page_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 已有将近三年之久，期间我已经重装 Arch 不下十次了，但是却一直没有记录自己是怎么装的。正好，这次我的笔记本拿去维修了，维修点表示需要重装系统，这意味着本子拿回来之后，我得重装我的 Arch，我也借此机会，记录一下自己 Arch 的重装过程。
<!-- more -->

# 准备工作

**特别提醒**：对于双系统玩家，强烈建议先安装 Windows，再去安装 Linux。

需要准备以下材料：

- 一台电脑（废话）；
- 网络连接（本文采用无线网络）；
- ArchLinux 启动盘一枚（我用的是 USB 启动介质）；
    - 听到 @dingyx99 大佬介绍了一个东西叫做 [Ventory](https://www.ventoy.net/cn/index.html)，可以不需要另外弄启动盘，直接在电脑划出一个分区就行了，后面试试。
- 一台具有网络浏览器的设备，方便在安装的时候查看 [ArchWiki](https://wiki.archlinux.org)上的[攻略](https://wiki.archlinux.org/title/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))；
    - 非必需，玩得转的童鞋也可以考虑按照攻略上的说法，使用安装环境的[Lynx](https://lynx.invisible-island.net/lynx_help/Lynx_users_guide.html)来浏览攻略。

同时（双系统玩家）不要忘记在 Windows 下用系统自带的“磁盘管理”工具提前为 Linux 准备好一块空白分区（不要格式化）。

# 安装基本系统

这一部分的内容，基本按照 [ArchWiki](https://wiki.archlinux.org)上的[攻略](https://wiki.archlinux.org/title/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 来做即可。

## 进入 Live 环境

插入安装介质，并在电脑**刚刚按下开机键**的时候设法进入启动菜单，选中你的安装介质，按回车。  
例如：我的笔记本是 ROG 的幻 14，安装介质是一枚金士顿的 U 盘，我就在开机键按下之后，屏幕刚刚开始显示败家眼的图标时按下 Esc 键（这个键对于不同设备可能不一样）进入启动菜单。  
注意：对于 Windows 10/11 用户，如果电脑屏幕下半部分出现了几个圆点转圈圈，那么恭喜你，等着 Windows 开机然后重启吧（悲

进入安装介质以后，会提示选择引导加载程序，直接默认（第一个）选项 `Arch Linux install medium` 回车就好。如果屏幕闪过大量文字后，清屏，显示一些 `Arch Linux...` 开头，`root@archiso ~ #` 结尾且后面有一个光标闪烁，恭喜你成功启动到 Live 环境。

## 检查引导模式

输入以下内容：

```sh
ls /sys/firmware/efi/efivars
```

如果命令正常输出了目录下的东西（可能会很多，占满整个屏幕），那么你的系统就是以 UEFI 模式引导的。否则，如果输出诸如 `No such file or directory` 之类的错误信息，你的系统可能是以 BIOS 等其他模式引导的。  
我的电脑下这个命令正常输出，因此判定我的系统以 UEFI 模式引导。下文的所有步骤，均基于这个结果来操作。

## 连接到互联网

幻 14 似乎没有网线接口，因此这里直接用无线网络\~  
使用 [iwd](https://wiki.archlinux.org/title/Iwd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 来连接到网络。输入 `iwctl`，如果屏幕显示 `[iwd]#` 则正常进入 iwd 的交互界面。输入 `device list`，即可查看所有可用的网络设备。我这里显示只有一个无线网卡 `wlan0`。下文中，将使用 `<device>`表示在这一步中你选用的无线设备。  
输入 `station <device> scan` 以使设备扫描可用的网络，输入 `station <device> get-networks` 来获取周围可用的 WiFi 的名称。选取你所希望使用的那个 WiFi，其名称记为 `<SSID>`，输入 `station <device> connect <SSID>`，并输入密码（如果有），即可连接到 WiFi，然后按 Ctrl-D 退出 iwd。  
建议在退出 iwd 之后使用 `ping` 验证一下网络连接：输入 `ping <URL> -c 3`，其中 `<URL>` 是你可以正常访问的网络的网址（不要带上 `http://` 或是 `https://`），如果输出包含形如 `xx bytes from xxx: icmp_seq=xxx ttl=xxx time=xxx` 的内容，说明网络连接正常，可以进入下一步骤。

## 更新系统时间

输入 `timedatectl set-ntp true`，并按下回车。

## 建立硬盘分区

建议事先阅读[官方教程的相关内容](https://wiki.archlinux.org/title/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%BB%BA%E7%AB%8B%E7%A1%AC%E7%9B%98%E5%88%86%E5%8C%BA)来了解大致信息。

**风险警告**：此步骤的误操作，可能导致电脑无法进入系统或者 Windows 系统及数据丢失等严重后果，请仔细阅读官方教程，并查阅其他资料，确保你知道自己在做什么！

输入 `lsblk` 可以查看各个存储设备和硬盘分区的简要情况，输入 `fdisk -l` 可以查看更加详细的内容。通过输出内容中各个分区/设备的大小，应该可以大致判断各个存储设备及其分区和其名称的对应关系。忽略 `rom`，`loop` 或者 `airoot` 结尾的设备，在其中找到你给 Linux 预留的空白存储空间，找到它对应的设备（例如，我的叫做`/dev/nvme0n1`），记为 `<disk>`，并留意其中是否已经包含一个类型为 `EFI System` 的分区（似乎 Windows 一般会自动创建这个分区）。  
输入 `fdisk <disk>`，输入 `n` 来利用预留的空白存储空间创建一个新的分区，Partition number 和 First sector 直接使用默认数值即可，Last sector 默认会用完剩下全部的可用空间（你也可以使用`+xK,M,G,T,P`来手动指定大小），然后会提示该分区的类型。如果不是 `Linux filesystem` 请输入 `t`，然后输入刚刚新建的分区的编号，最后输入`20`，将其类型改变为 `Linux filesystem`，输入 `w` 来保存刚刚的修改，并退出 fdisk。

## 格式化并挂载分区

**风险警告**：此步骤的误操作，可能导致电脑无法进入系统或者 Windows 系统及数据丢失等严重后果，请仔细阅读官方教程，并查阅其他资料，确保你知道自己在做什么！

找到刚刚用 fdisk 新建的分区（输入 `fdisk -l`，找到 `Linux filesystem` 类型的那个分区），其名字记为 `<part>`（我的电脑上，它是 `/dev/nvme0n1p7`），输入 `mkfs.ext4 <part>` 来格式化它。  
然后输入 `mount <part> /mnt` 来挂载它，并找到上一节中提到的类型为 `EFI System` 的分区（记为 `<efi-part>`，在我的电脑上它是 `/dev/nvme0n1p1`），依次输入 `mkdir /mnt/boot` 和 `mount <efi-part> /mnt/boot`。

## 安装系统到硬盘

### 更换软件包管理器的下载源

编辑文件 `/etc/pacman.d/mirrorlist`（新手推荐使用命令 `nano /etc/pacman.d/mirrorlist`），按照[TUNA 镜像站的说明](https://mirrors.tuna.tsinghua.edu.cn/help/archlinux/)，将 `Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch` 添加到刚刚的文件顶端（使用 nano 的话，直接把这段代码输入进去，然后按一下回车，Ctrl-O 保存，Ctrl-X 退出）。

### 安装系统及软件到硬盘

输入以下命令：

```sh
pacstrap /mnt linux linux-firmware base iwd networkmanager nano grub efibootmgr os-prober ntfs-3g sudo
```

然后耐心等待软件包安装完成。

## Fstab

输入以下命令：

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

然后再输入以下命令，查看生成文件是否正确：

```sh
cat /mnt/etc/fstab
```

## 进入新系统

输入以下命令：

```sh
arch-chroot /mnt
```

## 设置时区

依次输入以下命令：

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

## 本地化

**提示**：本步骤将会影响以后系统能否使用中文、会不会乱码等与本地化相关的事情。

输入命令 `nano /etc/locale.gen`，然后搜索，并去掉以下内容前的`#`：

```plain
en_US.UTF-8
en_SG.UTF-8
zh_CN.UTF-8
zh_CN.GBK
zh_TW.UTF-8
```

**提示**：nano 中搜索的快捷键是 Ctrl-W。

然后按 Ctrl-O 保存，Ctrl-X 退出，输入以下命令：

```sh
locale-gen
echo "LANG=en_SG.UTF-8" >> /etc/locale.conf
```

## 网络配置

选择一个你喜欢的主机名，记为`<hostname>`，然后执行命令：`echo <hostname> >> /etc/hostname`。  
编辑 `/etc/hosts`（`nano /etc/hosts`），输入以下内容：

```plain
127.0.0.1   localhost
::1         localhost
127.0.1.1   <hostname>
```

按照官方教程的说法，我们需要在此时完成剩余的网络配置，包括安装软件和进行网络配置。而我们已经在[安装系统及软件到硬盘](#安装系统及软件到硬盘)这步完成了软件安装，且 NetworkManager 无法在此时完成配置，因此直接跳过，至下一步骤。

## 修改 Root 密码

默认的 Root 密码是随机生成的，在此我们需要手动更改，否则退出 Live 环境以后，我们将无法登录至 Root 账户（对于没有新建其他账户的电脑，这意味着无法进入系统）。  
使用以下命令，按照提示修改 Root 密码（放心，不需要猜出原来的那个随机的密码）：

```sh
passwd
```

**警告**：如果比尔知道电脑的 Root 密码，那么理论上他可以对这台电脑做任何事，因此请保证你的 Root 密码有一定强度（但不要让你自己都记不住：稍后我们还要用）。

## 安装引导程序

**警告**：这个步骤至关重要，没能正确安装/配置引导程序可能导致 Linux，甚至可能是 Windows 系统无法启动。

我使用 [GRUB 2](https://wiki.archlinux.org/title/GRUB_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 作为引导程序。它的依赖程序已经在[安装系统及软件到硬盘](#安装系统及软件到硬盘)中安装完成。执行以下命令来将 GRUB 安装到 EFI 分区：

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

如果提示 `No error reported.` 则安装正常。

然后编辑文件 `/etc/default/grub` 并在文件结尾加上一行以下内容（来让 GRUB 探测 Windows 的存在）：

```plain
GRUB_DISABLE_OS_PROBER=false
```

然后运行以下命令生成 GRUB 主配置文件：

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

## 重启

按 Ctrl-D 退出新安装的系统，然后输入 `reboot`。不要忘记移除安装介质和在电脑启动的时候修改/选用正确的启动条目。  
重启后，输入账户名：`root` 和刚刚修改的 Root 密码即可登录到新系统。

至此，一个基本的 ArchLinux 操作系统就安装完成了——除了没有网络连接，以及图形化界面的支持。这些问题，我们将稍后逐步解决。

# 配置新系统

## 用户

日常直接使用 Root 账户是极其危险的一件事，因此我们需要创建普通用户来使用 ArchLinux。

### 新建用户

选择一个你喜欢的名字，记为`<username>`。  
创建用户（并为其创建主文件夹）：

```sh
useradd -m <username>
```

然后修改其密码：

```sh
passwd <username>
```

### 修改权限

将用户加入群组 `wheel`：

```sh
usermod -aG wheel <username>
```

然后允许该用户使用 `sudo`：首先运行命令`EDITOR=nano visudo`，然后找到行`# %wheel ALL=(ALL) ALL`，去掉前面的 `#`，保存，退出 nano。

## 连接到网络

使用 [NetworkManager](https://wiki.archlinux.org/title/NetworkManager_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))：首先启用相关的系统服务：

```sh
systemctl start NetworkManager.service
systemctl enable NetworkManager.service
```

然后查找附近的 WiFi：

```sh
nmcli device wifi list
```

假设要连接的 WiFi 名称为 `<SSID>`，密码为 `<password>`，连接至 WiFi：

```sh
nmcli device wifi connect <SSID> password <password>
```

或者该 WiFi 没有密码：

```sh
nmcli device wifi connect <SSID>
```

最后，测试连接：

```sh
ping baidu.com -c 3
```

## 安装桌面环境

我选择的桌面环境为：Xorg + SDDM + KDE Plasma。

### 安装显示驱动

先安装核显的驱动，保证能够正常进入桌面环境，然后再考虑独显的事情。  
我的 CPU 是 AMD 的，因此核显驱动选择 `xf86-video-amdgpu`：

```sh
pacman -S xf86-video-amdgpu
```

### 安装 Xorg

```sh
pacman -S xorg
```

### 安装 SDDM

```sh
pacman -S sddm sddm-kcm
```

### 安装 KDE Plasma 及其配套软件

```sh
pacman -S plasma kde-applications
```

然后自信重启。如果能够正常进入图形化的登陆界面，说明配置正常。

### 中文化桌面环境

直接在 KDE 自带的系统设置里面设置即可。重启生效。

### 其他

注意记得将“点击文件或文件夹时”设置为“选中它们”，否则可能会导致文件操作不符合习惯。

从此以后的配置改用管理员账户（而非 Root 账户），因此某些命令可能需要加上 `sudo`。

## AUR Helper

AUR 是 ArchLinux 的一大竞争力，很多优秀的软件都被放在 AUR 上。  
我使用的 AUR Helper 是 pikaur。它不在官方的软件仓库里面。具体安装过程和配置可参考[项目的 GitHub 页面](https://github.com/actionless/pikaur)。  
建议安装后额外运行一下：

```sh
sudo pacman -S asp python-pysocks
```

## 中文输入法

采用的方案为 [Fcitx5](https://wiki.archlinux.org/title/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 下的中文输入插件。

输入以下命令：

```sh
sudo pacman -S fcitx5-im fcitx5-qt fcitx5-gtk fcitx5-chinese-addons fcitx5-configtool fcitx5-pinyin-zhwiki
```

并将以下内容输入到文件 `~/.pam_environment` 中（如果没有，就新建一个）

```sh
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=@im=fcitx
INPUT_METHOD  DEFAULT=fcitx
SDL_IM_MODULE DEFAULT=fcitx
```

然后运行 fcitx5 配置，“添加输入法”里面找到拼音，点击“添加”。另外建议多看看其他按钮，看一下有些什么可以调教的。

如果重启电脑后发现输入法没有运行，“系统设置”->“开机与关机”->“添加”->“添加应用”，找到“Fcitx 5”，点确定。

## 双显卡切换

我的电脑是 AMD 的核显和 RTX 2060MQ 的独显，采用的是独显做大部分显示，仅需要时手动使用核显的思路。

采用的切换方案是 [PRIME](https://wiki.archlinux.org/title/PRIME)。

首先需要安装驱动，这里为了性能，使用 Nivida 的闭源驱动 `nvidia`：

```sh
sudo pacman -S nvidia nvidia-prime
```

另外可以考虑安装以下验证用的软件包 `mesa-demos`：

```sh
sudo pacman -S mesa-demos
```

重启电脑，现在输入这两个命令验证安装是否正确：
```sh
glxinfo | grep "OpenGL render" # OpenGL renderer string: AMD RENOIR (DRM 3.41.0, 5.13.13-arch1-1, LLVM 12.0.1)
prime-run glxinfo | grep "OpenGL render" # OpenGL renderer string: NVIDIA GeForce RTX 2060 with Max-Q Design/PCIe/SSE2
```

如果安装正确，以后启动应用时在其前面加上 `prime-run` 就可以切换到独显来渲染了。

## 蓝牙

参考[官方指南](https://wiki.archlinux.org/title/Bluetooth_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))。

安装软件包：

```sh
sudo pacman -S bluez bluez-utils bluedevil
```

然后启用蓝牙服务：

```sh
sudo systemctl start bluetooth.service
sudo systemctl enable bluetooth.service
```

此时电脑的状态栏会出现蓝牙的图标。  

对于蓝牙耳机，应该再额外执行以下命令：

```sh
sudo pacman -S pulseaudio-bluetooth
```