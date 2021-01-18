---
tags: archlinux linux
title: 安装ArchLinux系统
---

# 预先配置

## 连接WiFi网络

```bash
iwctl device list
iwctl station ${device} scan
iwctl station ${device} get-networks
iwctl station ${device} connect ${ssid}
```

## 测试网络连接并同步时间

```bash
ping -c 1 archlinux.org
timedatectl set-ntp true # 与Windows系统同时使用时无需配置
```

## 硬盘分区格式化并挂载

```bash
fdisk -l
cfdisk /dev/${disk}
mkfs.fat -F 32 /dev/${part_esp} # 仅UEFI模式需要
mkfs.ext4 /dev/${part_root}
mount /dev/${disk_root}
mkdir /mnt/boot # 仅UEFI模式需要
mount /dev/${part_esp} /mnt/boot # 仅UEFI模式需要
```

# 基本系统

## 安装软件

> 如使用AMD CPU需将`intel-ucode`替换成`amd-ucode`

```bash
vim /etc/pacman.d/mirrorlist
pacstrap /mnt base base-devel htop inetutils intel-ucode linux linux-firmware networkmanager ntfs-3g vim wget
genfstab -U /mnt >>/mnt/etc/fstab
arch-chroot /mnt
```

## 配置时区并设定硬件时间

```bash
ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock -w # 不与Windows系统同时使用时配置
hwclock -w -l # 与Windows系统同时使用时配置
```

## 设定语言与区域

* en_US.UTF-8 UTF-8

* zh_CN.GB18030 GB18030

* zh_CN.GBK GBK

* zh_CN.UTF-8 UTF-8

* zh_CN GB2312

* zh_TW.UTF-8 UTF-8

* zh_TW BIG5

```bash
vim /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 >/etc/locale.conf
```

## 设定主机名

```bash
echo ${hostname} >/etc/hostname
cat >/etc/hosts <<EOF
127.0.0.1 localhost
::1 localhost
EOF
```

## 设定自启服务

```bash
systemctl enable fstrim.timer # 使用固态硬盘时需要配置
systemctl enable NetworkManager
systemctl enable systemd-timesyncd # 与Windows系统同时使用时无需配置
```

## 设置root密码

```bash
passwd
```

## 安装引导

### BIOS

```bash
pacman -S grub os-prober
grub-install --target=i386-pc /dev/${disk}
grub-mkconfig -o /boot/grub/grub.cfg
```

### UEFI

```bash
bootctl install
echo default arch.conf >/boot/loader/loader.conf
cat >/boot/loader/entries/arch.conf <<EOF
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/${part_root}) rw
EOF
```

## 退出安装环境并重启

```bash
exit
reboot
```

# 图形环境

> 进入后采用`nmtui`命令配置网络

## 安装第三方软件源

```bash
cat >>/etc/pacman.conf <<EOF

[archlinuxcn]
Server = https://opentuna.cn/archlinuxcn/\$arch
EOF
pacman -Sy archlinuxcn-keyring
```

### 如验证失败进行如下操作

```bash
rm -fr /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinux
pacman -Sy archlinuxcn-keyring
```

## 安装软件

> 如`cat /proc/sys/kernel/random/entropy_avail`的值小于1000，请按[Random number generation - ArchWiki](https://wiki.archlinux.org/index.php/Random_number_generation)页面处理

```bash
pacman -S adobe-source-code-pro-fonts adobe-source-han-sans-cn-fonts adobe-source-han-sans-tw-fonts adobe-source-han-serif-cn-fonts adobe-source-han-serif-tw-fonts adobe-source-sans-pro-fonts bash-completion bdf-unifont cantarell-fonts code cups dina-font font-bh-ttf gentium-plus-font gnome gnome-shell-extension-topicons-plus-git gnu-free-fonts google-chrome ibus ibus-libpinyin inter-font jekyll noto-fonts opendesktop-fonts otf-fantasque-sans-mono otf-fira-mono tamsyn-font terminus-font tex-gyre-fonts ttf-anonymous-pro ttf-arphic-ukai ttf-arphic-uming ttf-bitstream-vera ttf-cascadia-code ttf-croscore ttf-dejavu ttf-droid ttf-fantasque-sans-mono ttf-fira-code ttf-fira-mono ttf-hack ttf-ibm-plex ttf-inconsolata ttf-jetbrains-mono ttf-junicode ttf-liberation ttf-linux-libertine ttf-monofur ttf-opensans ttf-roboto ttf-ubuntu-font-family ttf-wps-fonts vlc wechat-uos wireshark-qt wqy-bitmapfont wqy-microhei wqy-zenhei xorg-fonts-type1 yay
pacman -Rs epiphany gnome-books gnome-boxes gnome-contacts gnome-documents gnome-maps gnome-music gnome-photos gnome-software totem
yay -S ttf-ms-fonts wps-office-cn wps-office-mime-cn wps-office-mui-zh-cn
```

## 设定自启服务

```bash
systemctl enable cups.socket
systemctl enable gdm
```

## 创建新用户并设置密码和权限

```bash
useradd -m -s /bin/bash ${username}
passwd ${username}
EDITOR=vim visudo
```

> 切换到新建的用户

```bash
cp /etc/skel/.bash* ~
```
