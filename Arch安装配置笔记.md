---
title: Arch安装配置笔记
date: 2019-02-15 12:50:33
categories: Linux
tags:
- Arch
---

整理一下，方便以后安装Arch Linux

<!-- more -->

刻录u盘略过。。。

# 检测是否是UEFI启动

```
ls /sys/firmware/efi/efivars
```

- 文件不存在说明不是以UEFI启动

# 连接WIFI

```
wifi-menu
```

然后可以检测连接是否成功

```
ping -c 5 www.baidu.com
```

# 更新系统时间

### 操作

```
 timedatectl set-ntp true
```

查看服务状态

```
timedatectl status
```

### 原因

- 系统时间不对可能造成`ssl`连接失败导致安装出错

- 验证软件包签名，确定公钥过期没过期
- `https`证书，也要验过期时间

# 分区

### 检测分区情况

```
lsblk
```

### 分区

```
cfdisk /dev/sdx
```

![](Arch安装配置笔记/fenqu.png)

### 格式化分区

1. 格式化成`ext4`

```
mkfs.ext4 /dev/sdX2
```

2. 格式`swap`分区并激活

```
mkswap /dev/sdX3
swapon /dev/sdX3
```

### 挂载分区

1. 首先将根分区 [挂载](https://wiki.archlinux.org/index.php/Mount) 到 `/mnt`

```
mount /dev/sdX2 /mnt
```

（可选）

2. 挂载`home`分区

```
mkdir /mnt/home
mount /dev/sdx3 /mnt/home
```

3. 挂载`boot`分区(uefi)

```
mkdir -p /mnt/boot/efi
mount /dev/sdx4 /mnt/boot/efi
```

# 安装

### 选择镜像源

```
vi /etc/pacman.d/mirrorlist
```

在列表中越前的镜像在下载软件包时有越高的优先权，将清华源复制到第一行

### 安装基本系统

```
 pacstrap /mnt base base-devel
```

# 配置系统

### Fstab

用以下命令生成 [fstab](https://wiki.archlinux.org/index.php/Fstab) 文件 (用 `-U` 或 `-L` 选项设置UUID 或卷标)：

```
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

[Change root](https://wiki.archlinux.org/index.php/Change_root) 到新安装的系统：

```
arch-chroot /mnt
```

### 时区

设置时区

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

硬件时间设置，默认为`UTC`时间

```
hwclock --systohc 
```

### 本地化

`/etc/locale.gen` 是一个仅包含注释文档的文本文件。指定您需要的本地化类型，只需移除对应行前面的注释符号（`＃`）即可，建议选择带 `UTF-8` 的项

```
# nano /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```

接着执行以生成 locale 讯息

```
 locale-gen
 echo LANG=en_US.UTF-8 > /etc/locale.conf
```

### 网络

设置主机名

```
echo cc > /etc/hostname
```

添加对应的信息到`host`

```
# nano /etc/hosts
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

### Root密码

```
passwd
```


### 安装引导

```
pacman -S grub efibootmgr
```

- 非uefi

```
grub-install --target=i386-pc /dev/sdx 
#/dev/sdx 是已经完成分区的磁盘，grub 将安装到它上面。
```

- uefi

```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub
# 注意要挂载/boot/efi，见上面挂载boot分区那步
```

生成主配置文件

```
 grub-mkconfig -o /boot/grub/grub.cfg
```

### 安装微码

```
pacman -S amd-ucode
```

### 无线连接

```
pacman -S iw wpa_supplicant dialog
```

# 重启

```
exit #返回安装环境
umount -R /mnt
reboot
```

# 配置新系统

参考：[Linux下终极字体配置方案]([https://ohmyarch.github.io/2017/01/15/Linux%E4%B8%8B%E7%BB%88%E6%9E%81%E5%AD%97%E4%BD%93%E9%85%8D%E7%BD%AE%E6%96%B9%E6%A1%88/](https://ohmyarch.github.io/2017/01/15/Linux下终极字体配置方案/))

### 新建用户

```
useradd -m -G wheel cc
passwd cc
```

设置权限

```
nano /etc/sudoers
在 root ALL=(ALL) ALL 下面添加
用户名 ALL=(ALL) ALL
为你刚才创建的用户 添加sudo权限
```

### 添加archcn源

在 `/etc/pacman.conf` 文件末尾添加两行

```
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

然后请安装 `archlinuxcn-keyring` 包以导入` GPG key`。

### 常用软件

```shell
 pacman -S git make cmake openssh gcc g++ gdb vim wget
```

### 桌面环境

1. 安装`xorg`

   `Xorg`是`Linux`下的一个著名的开源图形服务，我们的桌面环境需要`Xorg`的支持。

   ```
   pacman -S xorg-server xorg-xinit
   ```

2. 桌面环境

   - `xfce`

     ```
     pacman -S xfce4 xfce4-goodies
     ```

   - `KDE(Plasma)`

     ```
      pacman -S plasma kde-applications
     ```
     更多:[archwiki](https://wiki.archlinux.org/index.php/Desktop_environment_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
3. 桌面管理器

   安装好了桌面环境包以后，我们需要安装一个图形化的桌面管理器来帮助我们登录并且选择我们使用的桌面环境，这里我推荐使用`sddm`。

   ```
   pacman -S sddm
   ```

4. 开机自启`sddm`服务

   ```
   systemctl enable sddm
   ```

### yay

Arch拥有一个强大的用户库AUR即Arch User Repository，为我们提供了官方包之外的各种软件包，一些闭源的软件包也可以在上面找到，可以说AUR极大地丰富了软件包的种类与数量，并可以配合yay这样的工具为用户省下大量安装、更新软件包的时间。

yay实际上也是一个软件包，我们可以把它看成是对pacman的包装，它兼容pacman的所有操作，最大的不同是我们可以用它方便地安装与管理AUR中的包，下面的许多软件包都是在AUR库中的，也都是使用AUR来安装的。

**安装**

```shell
pacman -S yay
```

### zsh

1. 安装

`pacman -S zsh`

2. 设置zsh为默认shell

`sudo chsh -s /bin/zsh username`

3. 安装oh-my-zsh

`yay -S oh-my-zsh-git`

4. 插件

- zsh-autosuggestions

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

- zsh-syntax-highlighting

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

5. 配置`~/.zshrc`

```she
plugins=( zsh-autosuggestions zsh-syntax-highlighting)
```

### 配置网络
1. 安装networkmanager

`pacman -S networkmanager`
2. 设置开机自启并启用

`systemctl enable NetworkManager`
`systemctl start NetworkManager`
3. 安装前端插件

- GTK3+前端小程序，工作在Xorg环境下，带有一个系统托盘。 
`pacman -S network-manager-applet`
- kde可以只安装plasma-nm，然后通过 面板的选项 > 添加部件 > 网络 来把它添加到KDE的任务栏上。

### 中文输入法
**搜狗输入法**

```shell
 pacman -S fcitx fcitx-im fcitx-sogoupinyin fcitx-configtool zenity
```


装完以后需要修改`/etc/profile`文件，在文件开头加入三行

```shell
export XMODIFIERS="@im=fcitx"
export GTK_IM_MODULE="fcitx"
export QT_IM_MODULE="fcitx"
```
可以解决一些软件无法调出fcitx的问题。

### 字体

1. 安装noto全系字体

`pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji ttf-monaco`

`ttf-monaco`:终端字体

2. 安装meslo字体

`yay -S ttf-meslo`

3. 拉取配置文件

```shell
wget https://github.com/ohmyarch/fontconfig-zh-cn/blob/master/fonts.conf ~/.fontconfig/fonts.conf
```

4. 设置dpi

用[CX CALC](http://pxcalc.com/)这个工具计算出对应你显示器分辨率的DPI值，然后再设置（可适当调高，本机100，可调107）

5. 设置字体

设置等宽字体为`Monospace`

其他为`noto Sans sjk sc`

6. 使GTK程序能够显示彩色Emoji

`yay -S cairo-coloredemoji`

7. 刷新缓存然后重启

`fc-cache --force --verbose`

### synaps

Synapse是一个快速的软件启动器，可以方便地查找安装的软件，设置快捷键使用再也不用找软件入口了。

```she
pacman -S synapse
```

### VirtualBox

`pacman -S virtualbox virtualbox-ext-vnc virtualbox-guest-iso virtualbox-host-modules-arch`
再去官网下载Oracle VM VirtualBox Extension Pack ，在设置中导入使用。安装windows的过程不在这里讲解，记得安装之后在windows内安装扩展客户端软件即可。

### 系统备份

`pacman -S timeshifts`

### 声卡

`pacman -S alsa-utils pulseaudio pulseaudio-bluetooth`
图形化
`pacman -S pavucontrol`

### 蓝牙

```shell
sudo pacman -S bluez bluez-utils
sudo systemctl start bluetooth.service
sudo systemctl enable bluetooth.service
```

图形界面使用`blueman`

```shell
pacman -S blueman
```

为了消除登录时蓝牙请求权限，创建`/etc/polkit-1/rules.d/81-blueman.rules`

```shell
polkit.addRule(function(action, subject) {
  if (action.id == "org.blueman.rfkill.setstate" && subject.local && subject.active && subject.isInGroup("wheel")) {
      return polkit.Result.YES;
  }
  if (action.id == "org.blueman.network.setup" && subject.local && subject.active && subject.isInGroup("wheel")) {
      return polkit.Result.YES;
  }
});
```

### 文本工具

` pacman -S foxitreader typora visual-studio-code-bin`

### 编译器

`pacman -S codeblocks`

### 互联网工具

`pacman -S firefox chrome filezilla teamviewer `

### 图形软件

`pacman -S flameshot`

### jdk8

1. 到[https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载jdk8

2. `yay -S jdk8`
3. 将下载的jdk8覆盖到` /tmp/yaourt-tmp-[your-user-name]/aur-jdk8`

4. `yay -S jdk8`

### 显卡

https://wiki.archlinux.org/index.php/Xorg

`pacman -S xf86-video-amdgpu mesa`

### 美化

![](Arch安装配置笔记/1.png)

1. 主题Materia KDE

https://github.com/PapirusDevelopmentTeam/materia-kde

``pacman -S materia-kde kvantum-theme-materia``

2. 图标papirus

https://github.com/PapirusDevelopmentTeam/papirus-icon-theme

``pacman -S papirus-icon-theme``

3. 光标mac--capitaine-cursors

https://github.com/keeferrourke/capitaine-cursors

`yay -S capitaine-cursors`

### KDE

```shell
pacman -S dolphin dolphin-plugins konsole
```

