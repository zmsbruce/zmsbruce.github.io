---
title: Arch Linux 安装与美化
data: 2024-09-27 10:35:02 +0800
categories: [Linux]
tags: [Arch Linux, Linux]
description: Windows 和 Arch Linux 双系统下 Arch Linux 的安装以及美化的详细步骤
author: zmsbruce
---

![](/assets/img/2024-09-27-Arch%20Linux安装与美化/cover.png)

## 写在开头

Arch Linux 是一个轻量、灵活、滚动更新的 Linux 发行版，适合有时间、有兴趣、热爱自由的读者。由于 Arch Linux 仅提供了一个很简洁的安装环境，用户有极大的空间根据自己的喜好进行安装和配置，因此初学者往往感到难以适应。本文记录了 Windows 和 Arch Linux 双系统下 Arch Linux 的安装以及美化的全流程，以便有需要的时候进行查看。

## 准备工作

### 获取安装镜像

安装镜像是我们安装 Arch Linux 的工具，其中包含了一个 Arch Linux live 系统。该系统功能十分强大，不仅可以用于安装系统，也可用于抢救系统。在 [Arch Linux 下载网站](https://archlinux.org/download/) 选择一种下载方式（推荐使用 China 列表下的镜像，下载速度更快），下载如下文件

```
archlinux-20xx.xx.xx-x86_64.iso
```

在下载之后，为确保文件没有损坏，需要计算文件的 SHA256 值，在 Windows 系统中，您可以打开 PowerShell 窗口，输入

```powershell
#请将 20xx.xx.xx 替换为镜像名中的日期
certutil -hashfile .\archlinux-20xx.xx.xx-x86_64.iso SHA256
```

在 [Arch Linux 下载网站](https://archlinux.org/download/) 中找出 Checksums 部分，对照其中的 SHA256 值，如果不相同，需要重新下载。

### 准备安装介质

使用 U 盘作为安装介质，制作工具推荐 UltraISO、 Rufus 和 Ventoy ，具体操作请自行查阅，都非常简单。

### 磁盘分区

在 Windows 下右键开始按钮，选择“磁盘管理”，对想要进行分区的磁盘右键选择“压缩卷”，选择合适的分区大小进行分区操作，推荐至少分配 50GiB 的空间。

### BIOS 设置

进入 BIOS，在“设置”或者“安全”选项卡中找到“安全启动”项，并将其设置为关闭，找到“允许 USB 启动”项，将其设置为开启。

## 安装系统

### 启动安装环境

将 U 盘插入您的电脑，重启电脑，连续按电脑的启动快捷键，直到出现启动列表，选择您的 U 盘。在随后弹出的 GRUB 引导页面中，选择第一项

```
Arch Linux install medium (x86_64, UEFI)
```

耐心等待，如果出现了“Welcome to Arch Linux”字样，说明安装环境启动成功了。

### 连接互联网

Arch Linux 镜像只提供一个基本的安装环境，软件包仍需要连接互联网后下载。首先确保网络接口已经启用。

```sh
ip link
```

以下面的输出为例，其中 `lo` 和 `wlp1s0` 就是您的网络接口名称，其尖括号内的 `UP` 表示接口已经启用。

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether e8:fb:1c:9e:57:d2 brd ff:ff:ff:ff:ff:ff
```

如果没有出现 `UP` 字样，则需要使用以下命令启用接口：

```sh
# 请将 XXX 替换为您的网络接口名称
ip link set XXX up
```

如果使用无线连接，请使用 `rfkill` 命令确认无线网卡没有被屏蔽。其中如果 `HARD` 列下的某一项为 `blocked` ，说明其被硬件屏蔽，而如果 `SOFT` 下的某一项为 `blocked` ，则说明其被软件屏蔽：

```
ID TYPE      DEVICE      SOFT      HARD
 0 wlan      phy0   unblocked unblocked
 2 bluetooth hci0   unblocked unblocked
```

如果无线网卡被硬件屏蔽，请使用硬件开关启用它，如果被软件屏蔽，请输入以下命令来启用：

```sh
rfkill unblock wlan
```

如果使用有线连接，只需要将网线插入到电脑相应接口即可连接，如果选择手机 USB 网络共享，将手机和电脑用数据线连接，然后在手机通知栏中选择“USB网络共享”即可连接。下面介绍连接 WiFi 的方法：

使用 `iwctl` 命令进入程序交互环境，在其中继续输入命令

```sh
device list
```

您可能会获得下面的输出：

```
                       Devices
------------------------------------------------------
Name   Address            Powered  Adapter  Mode
------------------------------------------------------
wlan0  xx:xx:xx:xx:xx:xx  on       phy0     station
```

继续输入命令

```sh
station wlan0 scan
station wlan0 get-networks
```

输出示例如下：

```
         Available Networks
------------------------------------
Network name    Security    Signal
------------------------------------
mySSID          psk         ****
```

继续输入命令

```sh
station wlan0 connect mySSID
```

接下来输入密码，如果没有任何报错，说明您已经连接成功了。接下来输入

```sh
quit
```

退出交互环境。在此之后，您可以使用 `ping` 来测试网络环境

```sh
ping https://archlinux.org
```

如果输出如下，则成功连上了互联网

```
PING www.archlinux.org (2a01:4f9:c010:6b1f::1) 56 bytes of data
64 bytes from archlinux.org (2a01:4f9:c010:6b1f::1): icmp_seq=1 ttl=41 time=264 ms
64 bytes from archlinux.org (2a01:4f9:c010:6b1f::1): icmp_seq=2 ttl=41 time=266 ms
64 bytes from archlinux.org (2a01:4f9:c010:6b1f::1): icmp_seq=3 ttl=41 time=271 ms
64 bytes from archlinux.org (2a01:4f9:c010:6b1f::1): icmp_seq=4 ttl=41 time=267 ms
^C
--- www.archlinux.org ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 264.381/267.294/271.263/2.498 ms
```

### 磁盘分区与挂载

#### 分区

首先，使用 `lsblk` 命令查看系统的分区情况，一个输出示例如下：

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 476.9G  0 disk 
├─nvme0n1p1 259:1    0   260M  0 part 
├─nvme0n1p2 259:2    0   128M  0 part 
├─nvme0n1p3 259:3    0 145.9G  0 part 
├─nvme0n1p4 259:4    0   644M  0 part 
├─nvme0n1p5 259:5    0 201.1G  0 part 
├─nvme0n1p6 259:6    0     1G  0 part 
└─nvme0n1p7 259:7    0   128G  0 part 
```

您可以根据之前分区的大小选择对应的磁盘（此处为 nvme0n1p7 分区，对应磁盘 nvme0n1），然后输入下面的命令进入分区界面

```sh
cfdisk /dev/nvme0n1
```

cfdisk 是图形化的分区指令，比 fdisk 简单很多。其下面有一行操作文件，通过左右方向键可以移动到不同选项。上下方向键可以选择不同分区进行操作。
- \[New\] 选项为新建分区，将方向键选择到未分配的分区，选择 \[New\] 选项，回车后会提示新分区大小，输入大小即可创建一个新的分区
- \[Quit\] 可以退出 cfdisk ，并且不保存修改，也就是之前所有没有写入的操作一律作废
- \[Help\] 选项可以查看 cfdisk 帮助
- \[Write\] 选项才是真的执行写入操作，使用后会对操作的磁盘执行写入，以前做的修改会生效
- \[Type\] 选项可以改变分区类型，boot 分区选择 EFI 分区类型，根分区选择 ext4 类型

以上面的磁盘分区为例，光标移动到 nvme0n1p7 处，使用 \[New\] 新建分区，并根据提示选择全部大小和 ext4 类型，再使用 \[Write\] 执行写入操作，最后使用 \[Quit\] 退出。

#### 建立交换文件

交换文件相当于 Windows 中的虚拟内存，也就是利用硬盘空间充当内存。当内存相对不足时，部分内存中的内容会交换到硬盘中，从而释放内存。交换文件的大小是个众说纷纭的问题，可以参考下面的表格选择交换文件的大小：

| 内存大小 | 推荐大小（使用休眠） | 推荐大小（不使用休眠） |
| :------: | :------------------: | :--------------------: |
|   2GiB   |       4096MiB        |        4096MiB         |
|   4GiB   |       5793MiB        |        5793MiB         |
|   8GiB   |       8192MiB        |        8192MiB         |
|  16GiB   |       11585MiB       |        8192MiB         |
|  32GiB   |       16384MiB       |        4096MiB         |
|  64GiB   |       23170MiB       |          0MiB          |

使用 `dd` 命令创建交换文件。例如，创建一块 8 GiB （=8192 MiB）大小的交换文件。

```sh
dd if=/dev/zero of=/mnt/swapfile bs=1M count=8192 status=progress
```

然后修改权限

```sh
chmod 0600 /mnt/swapfile
```

最后，格式化并启用交换文件

```sh
mkswap -U clear /mnt/swapfile
swapon /mnt/swapfile
```

#### 挂载

接下来，将根分区和 boot 分区挂载到不同的目录中

```sh
mount /dev/nvme0n1p7 /mnt
mount /dev/nvme0n1p1 /mnt/boot
```

> 如果您不知道哪个分区是引导分区，可以使用 `fdisk -l` 进行查看，其中类型为 EFI System 的即为引导分区。
{: .prompt-tip}

### 安装基础软件

#### 更新软件仓库

首先使用 reflector 选择合适的软件仓库镜像。输入

```sh
reflector -p https -c China --delay 3 --completion-percent 95 --sort score --save /etc/pacman.d/mirrorlist
```

之后，更新软件包

```sh
pacman -Syy
```

#### 安装基础软件包

首先使用 pacstrap 安装 base，linux 和 linux-firmware 三个软件包：

```sh
pacstrap -K /mnt base linux linux-firmware
```

如果报告“验证软件包错误”，可以尝试以下方法，然后重新安装：

```sh
pacman-key --init
pacman-key --populate
pacman -Sy archlinux-keyring
```

### 生成 fstab 文件

fstab 是一个系统文件，决定了系统启动时如何自动挂载分区。没有 fstab，系统将找不到根分区，从而无法启动。Arch Linux 提供了自动生成 fstab 的工具，我们利用它直接生成

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

### chroot 切换到系统

```sh
arch-chroot /mnt
```

### 设置时区

使用软链接设置时区

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

之后设置硬件时间

```sh
hwclock --systohc
```

### 设置 locale

使用文本编辑器（vim, nano）打开 `/etc/locale.gen` 文件，在其中找到 `en_US.UTF8` 和 `zh_CN.UTF8` 两行，将前面的 `#` 删除。之后，使用 `locale-gen` 命令生成 locale 。

之后，我们创建并编辑 `/etc/locale.conf` 文件，输入下面内容

```
LANG=en_US.UTF-8
```

### 网络配置

编辑 `/etc/hostname` 文件，设置主机名。之后，安装网络管理器

```sh
pacman -S networkmanager
```

NetworkManager 附带一个守护程序。在 Arch Linux 中，守护程序由 systemd 管理。我们使用 systemd 设置 NetworkManager 开机自动启动。

```sh
systemctl enable NetworkManager.service
```

### 设置 root 密码

使用 `passwd` 命令，根据提示设置密码，密码不会以 `*` 的形式呈现，这是正常现象。

### 安装引导

#### 安装微码

使用 Intel CPU：

```sh
pacman -S intel-ucode
```

使用 AMD CPU：

```sh
pacman -S amd-ucode
```

#### 安装引导加载程序

首先，安装必要的软件

```sh
pacman -S grub efibootmgr os-prober
```

之后安装 GRUB 到计算机：

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id="Arch Linux"
```

为了能检测到 Windows 系统，需要启用 os-prober 。打开 `/etc/default/grub` 文件，找到 `GRUB_DISABLE_OS_PROBER` 所在行，取消注释并将 true 改为 false 。

最后生成 GRUB 配置

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

### 安装显卡驱动

Intel 集显：

```sh
pacman -S mesa vulkan-intel intel-media-driver
```

AMD 集显或独显：

```sh
pacman -S mesa xf86-video-amdgpu vulkan-radeon libva-mesa-driver
```

> 有关 NVIDIA 显卡驱动的安装比较麻烦，且笔者并没有进行过实际操作，因此文章略过这部分内容。您可以通过 [这篇文章](https://zhuanlan.zhihu.com/p/568981775) 进行相关内容的查阅。
{: .prompt-warning}

### 安装桌面环境

#### 显示服务

Linux 的主流显示服务目前有两个：Xorg 和 Wayland 。Xorg 是 X 窗口系统（通常称为 X11 或 X）的公开开源实现，历史悠久稳定。而 Wayland 是一种较新的替代显示服务协议，它在安全、稳定和图形性能方面相较老旧的 Xorg 表现更出色。

#### 桌面环境

Linux 的主流桌面环境为 KDE Plasma（基于 Qt） 和 GNOME（基于 GTK）。GNOME 是大多数 Linux 发行版的默认桌面环境，而 Arch Linux 社区中更流行 KDE Plasma 。

#### 安装

GNOME：

```sh
pacman -S gnome gnome-tweaks gdm
systemctl enable gdm.service
```

KDE Plasma：

```sh
pacman -S plasma kde-applications
systemctl enable sddm.service
```

### 新建用户

我们已经使用 root 用户登录了系统，但是 root 用户的权限太高了，日常使用 root 可能带来安全风险。因此我们创建一个普通用户，并为其设置密码。

```sh
# 将 username 替换为你想要的用户名
useradd -m -G wheel <username>
passwd <username>
```

普通用户的权限又太低了，有时我们需要 root 权限进行系统管理，比如系统更新。这时我们需要提升权限。随后使用 nano 或 vim 编辑 /etc/sudoers 。由于该文件十分重要，所以我们不能使用以下命令编辑

```sh
vim /etc/sudoers
```

而必须通过 visudo 工具编辑。这个工具提供了语法检查，防止文件错误。

```sh
EDITOR=vim visudo
```

进入编辑器之后，请删除 “%wheel ALL=(ALL:ALL) ALL” 前面的 “#”。编辑完成后，我们使用以下命令切换身份到新创建的用户。

```sh
su - <username>
```

请注意，此时的命令提示符变成了“$”，这代表普通用户。

### 重启

输入下面命令退出 chroot 环境，取消挂载并重启电脑：

```sh
exit
umount -R /mnt
reboot
```

至此，Arch Linux 系统就安装完毕了。下一篇带来的是有关 Arch Linux 的美化，以让用户达到更良好的使用体验。

## AUR 介绍和使用

Arch Linux 的一大魅力所在就是其 AUR 仓库。Arch 用户软件仓库（Arch User Repository，AUR）是为用户而建、由用户主导的 Arch 软件仓库。AUR 中的软件包以软件包生成脚本（PKGBUILD）的形式提供，用户自己通过 makepkg 生成包，再由 pacman 安装。创建 AUR 的初衷是方便用户维护和分享新软件包，并由官方定期从中挑选软件包进入 community 仓库。

AUR 不以构建好的二进制软件包为主，有大量软件包需要您在本地编译才能安装使用。如果想要安装和使用 AUR ，首先安装 base-devel 库以便进行编译

```sh
sudo pacman -S base-devel
```

一般地，使用 AUR 进行应用安装的方法如下（以 [visual-studio-code-bin](https://aur.archlinux.org/packages/visual-studio-code-bin) 为例）：

1. 在 AUR 网站 [https://aur.archlinux.org](https://aur.archlinux.org) 上进行相应软件包的搜索；
2. 选择对应的软件，进入详情页，复制 Git Clone URL 下的网址 [https://aur.archlinux.org/visual-studio-code-bin.git](https://aur.archlinux.org/visual-studio-code-bin.git)；
3. 在本地运行指令：`git clone https://aur.archlinux.org/visual-studio-code-bin.git`，将包下载到本地；
4. 打开 visual-studio-code-bin 文件夹，其下面应该存在一个 PKGBUILD 文件，在该文件夹下运行 `makepkg -si` 即可。有关 Makepkg 的使用，请参考 [Makepkg-ArchWiki](https://wiki.archlinux.org/title/Makepkg)；

为了获得更好的体验，推荐安装 AUR 助手，常见的 AUR 助手包括 paru 和 yay 。paru 和 yay 都是 pacman 的封装，pacman 支持的选项和参数它们都支持。它们还有额外的功能，包括管理 AUR 仓库中的软件包。以 paru 为例，在命令行中输入

```sh
sudo pacman -S paru
```

安装 paru，之后如果想安装 visual-studio-code-bin 的 AUR 包，只需要输入

```sh
paru -S visual-studio-code-bin
```

根据提示，即可完成安装。

## 国内环境使用优化

### 启用 archlinuxcn 仓库

archlinuxcn 仓库是由 Arch Linux 中文社区驱动的非官方软件仓库，包含许多官方仓库未提供的额外的软件包，以及已有软件的 git 版本等变种，提供了大量适合国人使用的软件包。在 `/etc/pacman.conf` 中，添加

```
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

之后，同步软件数据库，并安装 archlinuxcn-keyring

```sh
sudo pacman -Syyu
sudo pacman -S archlinuxcn-keyring
```

### 安装中文输入法

安装 `ibus-rime` 输入法和 `noto-fonts-cjk` 中文字体：

```sh
paru -S ibus-rime noto-fonts-cjk
```

重启系统，之后在键盘-输入源中去掉原来的输入源，添加`中文（Rime）`这一选项。在 `~/.config/ibus/rime` 目录下，新建 default.custom.yaml ，输入

```yaml
patch:
  schema_list:
    - schema: luna_pinyin_simp
```

之后，在通知栏的 Rime 图标处右键，点击`部署`，即可使用。如果您使用 Visual Studio Code，那么其呼出终端的按键组合 Ctrl+` 会被 Rime 占用。若要避免这一现象，在 default.custom.yaml 中添加

```yaml
switcher:
  hotkeys:
    - F4
```

之后重新部署。此外，如果您偏好英文作为首选输入方式（这在控制台操作时显得尤其常见），您可以在刚才的目录下，新建 luna_pinyin.custom.yaml ，输入

```yaml
switches:
  - name: ascii_mode
    reset: 1
    states: ["中文", "西文"]
```

之后仍然是重新部署。

### 科学上网

#### V2ray 和 QV2ray

```sh
paru -S v2ray qv2ray
```

#### Clash for windows

```sh
paru -S clash-for-windows-bin
```

## 美化

### Gnome 桌面美化

> 桌面美化和后面的美化部分具有一定程度的主观性，下面的内容仅供参考。
{: .prompt-warning}

在浏览器安装 GNOME Shell 集成插件（Chrome, Firefox 和 Edge 均支持）；在本地安装 `gnome-browser-connector` 之后，您便可以在浏览器的 [GNOME Shell Extenstions 网页](https://extensions.gnome.org/) 安装 GNOME 插件。笔者使用的插件有：

- AppIndicator and KStatusNotifierItem Support：对传统托盘图标的支持；
- Dash to Dock：它将默认 Dash 从 overview 中移出，并在 Dock 中对其进行转换，以便更轻松地启动应用程序；
- Proxy Switcher：在快捷菜单中选择代理模式；
- Transparent Top Bar (Adjustable transparency)：在顶部栏自由浮动时变为透明；

### 终端美化

安装并启用 zsh ：

```sh
paru -S zsh
chsh -s /bin/zsh
```

在重启后，终端就变为了 zsh 。之后，我们安装 [Oh My Zsh](https://ohmyz.sh/) 用于管理 Zsh 配置。它捆绑了数以千计的有用功能、帮助程序、插件、以及主题。

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

之后，我们安装 zsh 插件，包括 [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) （在您键入时根据历史记录和完成情况建议命令）、[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)（为 shell zsh 提供语法高亮显示）：

```sh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

安装完毕后，我们在 ~/.zshrc 内启用包括上述两个插件在内的一些有用的插件。我们找到 `plugins` 所在行，将其内容修改为：

```
plugins = (
    git
    sudo
    zsh-syntax-highlighting
    zsh-autosuggestions
)
```

其中，sudo 插件可以在双击 `esc` 后自动在命令前面添加或删除 sudo 字段，非常实用。完成后我们在终端中刷新配置即可启用：

```sh
source ~/.zshrc
```

此外，我们可以安装 [powerlevel10k](https://github.com/romkatv/powerlevel10k) 主题对终端进行美化。首先，您需要将终端字体改为某种 Nerd Font 以便显示图标，之后我们像安装之前两个插件一样安装主题

```sh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

之后在 ~/.zshrc 中找到 `ZSH_THEME` 字样，修改为

```
ZSH_THEME="powerlevel10k/powerlevel10k"
```

之后运行

```sh
source ~/.zshrc
```

这时会进入 powerlevel10k 的配置界面，根据操作一步步执行即可。

### GRUB 美化

在 [gnome-look](https://www.gnome-look.org) 下寻找一款您喜欢的 GRUB 主题（此处以 Vimix 为例），下载并解压，进入到目录中。Vimix 中存在一个安装脚本 install.sh，您可以直接运行该脚本进行安装。如果没有，则需要手动进行安装，首先将主题文件夹放置到系统的 grub 主题文件夹中

```sh
sudo cp -rf ~/Downloads/Vimix-4k /usr/share/grub/themes/Vimix
```

之后修改 /etc/default/grub 文件，找到 `GRUB_THEME` 一行，去掉前面的注释，修改为

```
GRUB_THEME="/usr/share/grub/themes/Vimix/theme.txt"
```

在执行安装脚本或手动安装后，需要更新配置，输入

```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

> 如果不想在 GRUB 页面中显示中文，您需要将系统语言修改回英文并重新启动。
{: .prompt-tip}


如果您觉得 GRUB 页面中的字体不够美观，您可以进行修改。首先，您需要制作 GRUB 渲染需要的 pf2 字体，以DejaVuSansMNerdFont-Regular ，字号为 32 为例：

```sh
sudo grub-mkfont 
-v \
--output=/usr/share/grub/themes/Vimix/dejavu-32.pf2 \
--size=32 \
/usr/share/fonts/TTF/DejaVuSansMNerdFont-Regular.ttf
```

命令会输出下面的内容

```
Font name: DejaVuSansM Nerd Font Regular 32
Max width: 39
Max height: 38
Font ascent: 33
Font descent: 12
Number of glyph: 13536
```

之后打开 /usr/share/grub/themes/Vimix/theme.txt 文件，修改对应的 item_font 内容为前面的 Font name 。之后更新 GRUB 配置即可。

### 启动过程界面美化

您可以使用 plymouth 来对启动界面进行美化。首先，您需要修改 `/etc/default/grub` 配置文件，找到 `GRUB_CMDLINE_LINUX_DEFAULT` 一行，修改为

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

之后更新 GRUB 配置。接下来我们安装 plymouth 和笔者喜欢的主题（可以换成您喜欢的主题）

```sh
paru -S plymouth plymouth-theme-arch-charge-big
```

安装完之后，我们首先修改 mkinitcpio 的 HOOKS，mkinitcpio 是 Arch Linux 用于生成 Linux 系统 初始内存文件系统（Initial RAM Filesystem，initramfs）的工具，帮助系统在启动时加载必要的模块和驱动程序。我们打开 `/etc/mkinitcpio.conf` 文件，将其中的 HOOKS 修改为：

```
HOOKS=(... plymouth ...)
```

注意，如果您使用 systemd HOOK，您需要将其放在 plymouth 前面。此外，如果使用 crypt HOOK，需要放在 plymouth 后面。之后我们执行

```sh
sudo mkinitcpio -P
```

为所有已安装的内核生成或更新 initramfs 镜像文件。

我们修改主题为 arch-charge-big ：

```sh
plymouth-set-default-theme -R arch-charge-big
```

之后，我们重启系统，便可以看到启动界面变成来 Arch Linux 的 logo 了。

