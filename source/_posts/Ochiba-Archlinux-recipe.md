---
title: Ochiba's Archlinux Recipe
tags:
  - archlinux
categories:
  - 技术宅的日常
date: 2020-09-05 00:30:32
---

<!--在此处插入头图-->

{%asset_img header.jpg "Cover"%}

<!--在此处插入概述-->
随着Windows对用户隐私的不断侵犯，如果需要一个安全隐私的计算平台，linux是不二之选。其中，Archlinux虽然安装复杂，但是其包管理器pacman和用户仓库yay内容十分丰富，使日常使用非常顺畅。本文记录了我自己安装archlinux的过程。

<!--more-->

<!--以下为正文-->

## Preface
- This is a record of the process of installing Arch Linux on a typical computer with `UEFI` support. If your computer don't support `UEFI`, the making partition part might be different.
### What you need:
- A computer to install Arch Linux on connected to internet (`target`);
- A `flash drive`, preferrably > 8 GiB;
- Another computer that's connected to the internet (`computer B`)


## Enter Arch Linux Installation Environment
### Prepare installation media
#### On `computer B`:
- Download arch linux mirror at [Arch Linux Offical Site](www.archlinux.org);
- Use tools such as `ventoy` or `rufus` to make a bootable USB of Arch Linux Installation media.
#### On `target`:
- Backup all files required;
- Go to system firmware and
    - Change boot mode to `UEFI`
    - Calibrate `firmware time` with `UTC`;
- Disconnect all irrelevant disks other than OS disks if possible;
- Connect `target` to internet;
- Boot the `target` with the `flash drive` made previously.
- Every step afterwards happen on `target`.

## Installation Environment Time Setup
- Turn ntp on: ```timedatectl set-ntp true```;
- Check the service status: ```timedatectl status```.

## Create and Mount Partitions
> The following commands would wipe your drive. Ensure everything important on `target` is backed up.
- List disks using `fdisk` and identify the OS disk on target: `fdisk -l`. 
    - The OS disk is usually `/dev/sda` if it's the only one in existence.
- Use fdisk to create new partition table: 
    - Enter fdisk: `fdisk /dev/sda`;
    - Create a new empty GPT partition table: press `g` and `[ENTER]`;
    - Create a new `EFI system partition` that's at least 260 MB: `n` / `[ENTER]`/ `[ENTER]`/ `+512M`/ `[ENTER]`/ 
    - Create a new `Linux Swap` that's at least 512 MB: `n` / `[ENTER]`/ `[ENTER]`/ `+1G`/ `[ENTER]`/ 
    - Create a new `Linux x86-64 root` that's whatsever left on your drive large: `n` / `[ENTER]`/ `[ENTER]`/ `[ENTER]`
    - Change `EFI system partition` type: `t`/ `1`/ `1`/ `[ENTER]`
    - Change `Linux Swap` type: `t`/ `2`/ `19`/ `[ENTER]`
    - Write changes to disk: `w`
- Format these partitions
    - Format EFI system partition to FAT32: ```mkfs.fat -F 32 /dev/sda1```
    - Format Linux x86-64 root to ext4: ```mkfs.ext4 /dev/sda3```
    - Initialize Linux Swap:  
        ```mkswap /dev/swap_partition```  
        ```swapon /dev/swap_partition```
- Mount Linux x86-64 root: ```mount /dev/sda3 /mnt```

## Install base system
- Run pacstrap script: ```pacstrap /mnt base linux linux-firmware```
- Generate an fstab file: ```genfstab -U /mnt >> /mnt/etc/fstab```

## Use chroot to change root environment to the installed system
> From now on root is changed to the new system. If for some reason you need to go back to a recover environment, also use this command.
- Change root: ```arch-chroot /mnt```

## Install GRUB
- Install GRUB and nano the text editor: ```pacman -S grub efibootmgr nano intel-ucode amd-ucode```;
- Create mounting point of EFI system partition: ```mkdir efi```;
- Mount EFI system partition: ```mount /dev/sda1 /efi```
- Install GRUB: ```grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB```
- Generate grub.cfg: ```grub-mkconfig -o /boot/grub/grub.cfg```

## System Time and Locale related tasks
> This section setups system locale. It's set to en-GB to prevent charset issues in tty. For Chinese user interface, you should setup USER LOCALE, NOT system locale.
- Set timezone:
    ```ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime```  
    ```hwclock --systohc```
- Set locale: Edit `/etc/locale.gen` and uncomment `en-GB.UTF-8 UTF-8`
- Generate locale: ```locale-gen```

## Network related tasks
- Install NetworkManager and iwd: ```pacman -S iwd networkmanager```
### If wired and DHCP:
- Find your network interface: ```ip address show```
    - In the following example output, network interface is `eth1`:
        ```
        12: eth1: <BROADCAST,MULTICAST,UP> mtu 1500 group default qlen 1                                                            link/ether 00:50:56:c0:00:01                                                                                            inet 192.168.88.1/24 brd 192.168.88.255 scope global dynamic                                                               valid_lft forever preferred_lft forever                                                                              inet6 fe80::79c6:bc71:aa49:fbf6/64 scope link dynamic                                                                      valid_lft forever preferred_lft forever                     
        ```

- Set Hostname: ```[your hostname] > /etc/hostname```
- Modify `hosts file`: edit `/etc/hosts` with:
    ```
    127.0.0.1	localhost
    ::1		localhost
    127.0.1.1	[your hostname].localdomain	[your hostname]
    ```
- Set configuration file: modify `/etc/systemd/network/20-wired.network` with:
    ```
    [Match]
    Name=[eth1] # Change [eth1] with your network interface

    [Network]
    DHCP=yes
    ```
- Enable network manager: ```systemctl enable NetworkManager```
### If wireless:
- Start `iwctl`: ```iwctl```
- Find your wlan card ID: ```device list```
- Scan for networks: ```station wlan0 scan```
- List all avaliable networks: ```station wlan0 get-networks```
- Connect to network: ```station wlan0e connect my_wifi```
- Enable iwd: ```systemctl enable iwd.service```

## User related tasks
- Set root password: ```passwd```;
- Install sudo and nano:  ```pacman -S sudo```;
- Setup sudo: Edit file file and uncomment to allow all memners of group wheel to execute any command
    - run ```EDITOR=nano visudo```;
    - Uncomment line `# %wheel ALL=(ALL) ALL`
- Create a user account: (Change `ochiba` to your preferred username): ```useradd -m -G wheel ochiba```;
- Set a password for the user (Change `ochiba` to your preferred username): ```passwd ochiba```.

## Install Desktop Environment
- Install xorg:  ```pacman -S xorg```;
- Install KDE Plasma:  ```pacman -S kde-applications plasma sddm```;
- Enable display manager: ```systamctl enable sddm```.

## Boot into Arch Linux
- Exit chroot mode: ```exit```;
- Shutdown `target`;
- Reconnect other disks if you disconected them in preparation stage;
- Remove `flash drive`;
- Start `target`, you'll be on Arch desktop.

## Post Install
### Thunderbolt
- Where thunderbolt is applicable: ```sudo pacman -S bolt```
- Obtain device UID: `boltctl info`
- Allow automatic connection: `boltctl enroll --policy auto [thunderbolts设备 UID]`
- Disconnect and connect thunderbolt device.
### Allow mounting other local drives:
- Login as root: ```su root```
- 

###  Localization
- Install Chinese fonts: ```pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji adobe-source-han-sans-otc-fonts wqy-microhei wqy-zenhei```
- Change user-level locale: Modify `~/.config/locale.conf` to:  
    ```
    LANG=zh_CN.UTF-8
    LC_CTYPE="zh_CN.UTF-8"
    LC_NUMERIC="zh_CN.UTF-8"
    LC_TIME="zh_CN.UTF-8"
    LC_COLLATE="zh_CN.UTF-8"
    LC_MONETARY="zh_CN.UTF-8"
    LC_MESSAGES="zh_CN.UTF-8"
    LC_PAPER="zh_CN.UTF-8"
    LC_NAME="zh_CN.UTF-8"
    LC_ADDRESS="zh_CN.UTF-8"
    LC_TELEPHONE="zh_CN.UTF-8"
    LC_MEASUREMENT="zh_CN.UTF-8"
    LC_IDENTIFICATION="zh_CN.UTF-8"
    LC_ALL=
    ```
- Add Archlinuxcn source:
    - Append `/etc/pacman.conf` with:
        ```
        [archlinuxcn]
        Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
        ```
- Enable Multilib:
    - Uncomment the following in `/etc/pacman.conf`:
        ```
        [multilib]
        Include = /etc/pacman.d/mirrorlist
        ```
    - Update: ```sudo pacman -Syu```
    - Install `archlinuxcn-keyring` package: ```sudo pacman -S archlinuxcn-ring```
- Install Chinese IME
    - Install  `fcitx5` framework and relevant IMEs`: ```sudo pacman -S fcitx5-im fcitx5-rime```
    - Append `~/.pam_environment` with:
        ```
        INPUT_METHOD  DEFAULT=fcitx5
        GTK_IM_MODULE DEFAULT=fcitx5
        QT_IM_MODULE  DEFAULT=fcitx5
        XMODIFIERS    DEFAULT=\@im=fcitx5
        ```
    - Copy `self-start batch` to `~/.config/autostart`: ```cp /usr/share/applications/fcitx5.desktop ~/.config/autostart/```
    - Make the script executable: ```chmod +x ~/.config/autostart/fcitx5.desktop```
- Set correct display fonts
    - Install fonts:```sudo pacman -S ttf-roboto noto-fonts noto-fonts-cjk adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts ttf-dejavu```
    - Modify `/etc/fonts/local.conf` as follows:
        ```xml
        <?xml version="1.0"?>
        <!DOCTYPE fontconfig SYSTEM "fonts.dtd">
        <fontconfig>
        <its:rules xmlns:its="http://www.w3.org/2005/11/its" version="1.0">
            <its:translateRule
            translate="no"
            selector="/fontconfig/*[not(self::description)]"
            />
        </its:rules>

        <description>Android Font Config</description>

        <!-- Font directory list -->

        <dir>/usr/share/fonts</dir>
        <dir>/usr/local/share/fonts</dir>
        <dir prefix="xdg">fonts</dir>
        <!-- the following element will be removed in the future -->
        <dir>~/.fonts</dir>

        <!-- 关闭内嵌点阵字体 -->
        <match target="font">
            <edit name="embeddedbitmap" mode="assign">
            <bool>false</bool>
            </edit>
        </match>

        <!-- 英文默认字体使用 Roboto 和 Noto Serif ,终端使用 DejaVu Sans Mono. -->
        <match>
            <test qual="any" name="family">
            <string>serif</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
            <string>Noto Serif</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
            <string>sans-serif</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
            <string>Roboto</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
            <string>monospace</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
            <string>DejaVu Sans Mono</string>
            </edit>
        </match>

        <!-- 中文默认字体使用思源黑体和思源宋体,不使用　Noto Sans CJK SC 是因为这个字体会在特定情况下显示片假字. -->
        <match>
            <test name="lang" compare="contains">
            <string>zh</string>
            </test>
            <test name="family">
            <string>serif</string>
            </test>
            <edit name="family" mode="prepend">
            <string>Source Han Serif CN</string>
            </edit>
        </match>
        <match>
            <test name="lang" compare="contains">
            <string>zh</string>
            </test>
            <test name="family">
            <string>sans-serif</string>
            </test>
            <edit name="family" mode="prepend">
            <string>Source Han Sans CN</string>
            </edit>
        </match>
        <match>
            <test name="lang" compare="contains">
            <string>zh</string>
            </test>
            <test name="family">
            <string>monospace</string>
            </test>
            <edit name="family" mode="prepend">
            <string>Noto Sans Mono CJK SC</string>
            </edit>
        </match>

        <!-- Windows & Linux Chinese fonts. -->
        <!-- 把所有常见的中文字体映射到思源黑体和思源宋体，这样当这些字体未安装时会使用思源黑体和思源宋体.
        解决特定程序指定使用某字体，并且在字体不存在情况下不会使用fallback字体导致中文显示不正常的情况. -->
        <match target="pattern">
            <test qual="any" name="family">
            <string>WenQuanYi Zen Hei</string>
            </test>
            <edit name="family" mode="assign" binding="same">
            <string>Source Han Sans CN</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
            <string>WenQuanYi Micro Hei</string>
            </test>
            <edit name="family" mode="assign" binding="same">
            <string>Source Han Sans CN</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
            <string>WenQuanYi Micro Hei Light</string>
            </test>
            <edit name="family" mode="assign" binding="same">
            <string>Source Han Sans CN</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
            <string>Microsoft YaHei</string>
            </test>
            <edit name="family" mode="assign" binding="same">
            <string>Source Han Sans CN</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
            <string>SimHei</string>
            </test>
            <edit name="family" mode="assign" binding="same">
            <string>Source Han Sans CN</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
            <string>SimSun</string>
            </test>
            <edit name="family" mode="assign" binding="same">
            <string>Source Han Serif CN</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
            <string>SimSun-18030</string>
            </test>
            <edit name="family" mode="assign" binding="same">
            <string>Source Han Serif CN</string>
            </edit>
        </match>

        <!-- Load local system customization file -->
        <include ignore_missing="yes">conf.d</include>

        <!-- Font cache directory list -->

        <cachedir>/var/cache/fontconfig</cachedir>
        <cachedir prefix="xdg">fontconfig</cachedir>
        <!-- the following element will be removed in the future -->
        <cachedir>~/.fontconfig</cachedir>

        <config>
            <!-- Rescan configuration every 30 seconds when FcFontSetList is called -->
            <rescan>
            <int>30</int>
            </rescan>
        </config>
        </fontconfig>
        ```

### Some other life quality improvements
- Install `yay`, the AUR automation tool
- Install `tlp` when using a laptop

## Reference:
- [Installation Guide](https://wiki.archlinux.org/index.php/installation_guide)
- [给 GNU/Linux 萌新的 Arch Linux 安装指南 rev.B](https://blog.yoitsu.moe/arch-linux/installing_arch_linux_for_complete_newbies.html)
- [Arch Linux CN 源使用帮助](https://mirrors.ustc.edu.cn/help/archlinuxcn.html)
- [Fcitx5 (简体中文)](https://wiki.archlinux.org/index.php/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [GRUB-Archwiki](https://wiki.archlinux.org/index.php/GRUB)
- [systemd-networkd-Archwiki](https://wiki.archlinux.org/index.php/Systemd-networkd)
- [Network configuration-Archwiki](https://wiki.archlinux.org/index.php/Network_configuration#Network_interfaces)
- [Font Configuration (简体中文)/Chinese (简体中文)](https://wiki.archlinux.org/index.php/Font_Configuration_(简体中文)/Chinese_(简体中文))