# Table of Contents

* [Preparing the disks](#preparing-the-disks)
* [Stage 3](#stage-3)
* [Installing the Gentoo base system](#installing-the-gentoo-base-system)
* [Kernel configuration](#kernel-configuration)
* [System configuration](#system-configuration)
* [System tools](#system-tools)
* [Bootloader](#bootloader)
* [References](#references)

---

Follow the guide: https://wiki.gentoo.org/wiki/Handbook:AMD64

# Preparing the disks

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks

GPT + UEFI, we want to keep it as close to defaults as possible
```bash
sudo fdisk /dev/sda
sudo mkfs.vfat -F 32 /dev/sda1
sudo mkfs.ext4 /dev/sda2
```

Mount the partitions
```bash
sudo mkdir /mnt/gentoo
sudo mount /dev/sda2 /mnt/gentoo
```

# Stage 3

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage

```bash
cd /mnt/gentoo
sudo wget https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20210616T214502Z/stage3-amd64-systemd-20210616T214502Z.tar.xz
sudo tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
sudo nano -w etc/portage/make.conf
```

Just add `-march=native` to `COMMON_FLAGS` and we're good to go  
And set `-j2` for `MAKEOPTS`

# Installing the Gentoo base system

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base

```bash
sudo mkdir --parents etc/portage/repos.conf
sudo cp usr/share/portage/config/repos.conf etc/portage/repos.conf/gentoo.conf
sudo cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

mount
```bash
sudo mount --types proc /proc /mnt/gentoo/proc
sudo mount --rbind /sys /mnt/gentoo/sys
sudo mount --make-rslave /mnt/gentoo/sys
sudo mount --rbind /dev /mnt/gentoo/dev
sudo mount --make-rslave /mnt/gentoo/dev
```

chroot
```bash
sudo chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

Now, time to work.  

Set the desktop/plasma/systemd profile
```bash
mount /dev/sda1 /boot
emrerge-webrsync
eselect news list
eselect news read
eselect profile list
eselect profile set 9
```

And go to sleep ...
```bash
emerge --ask --verbose --update --deep --newuse @world
```

Post emerge
```bash
emerge -a gentoolkit
ln -sf ../usr/share/zoneinfo/Europe/Paris /etc/localtime
nano -w /etc/locale.gen
```

Add the following
```
en_US.UTF-8 UTF-8
fr_FR.UTF-8 UTF-8
C.UTF8 UTF-8
```

Then
```bash
locale-gen
locale -a
eselect locale list
eselect locale set 4
```

To manually set locale definitions edit `/etc/locale.conf`  
See: https://wiki.gentoo.org/wiki/Localization/Guide

Finally
```bash
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

# Kernel configuration

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel

Install precompiled distribution kernel
```bash
emerge --ask sys-kernel/gentoo-kernel-bin
emerge -a linux-firmware
emerge --config sys-kernel/gentoo-kernel-bin:5.10.38
```

# System configuration

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/System

fstab
```bash
nano -w /etc/fstab
```

Trying this out, the most minimal *working* setup possible, I think.
```
#<device>                                       <dir>   <type>  <options>       <dump>  <fsck>
UUID="f279f7af-f31b-4318-9965-b973c159a3dd"     /       ext4    noatime,discard 0       1
UUID="BB72-E032"                                /boot   vfat    defaults        0       2
```

```bash
nano /etc/security/passwdqc.conf
```
Set `enforce` to `none`

# System tools

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Tools

```bash
emerge -a cronie mlocate dosfstools
```

# Bootloader

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader

GRUB
```bash
emerge --ask --verbose sys-boot/grub:2
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```

I really don't know what I'm doing at this point, fingers crossed.
```bash
emerge -a networkmanager plasma-desktop kdecore-meta firefox-bin
```

Eventually get `qterminal`

add user
```bash
useradd -m -G users,wheel,audio -s /bin/bash ismo
passwd ismo
```

Peace out
```bash
exit
cd
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -R /mnt/gentoo
reboot
```

---

## ???

systemd
```bash
systemd-machine-id-setup
hostnamectl set-hostname gentoo
emerge -a plasma-meta
```
---

## References

Mastering Markdown: https://guides.github.com/features/mastering-markdown/  
Gentoo cheat sheet: https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet  
EFI System Partition: https://forums.gentoo.org/viewtopic-t-1123855.html  
EFI: https://wiki.gentoo.org/wiki/User:Sakaki/Sakaki%27s_EFI_Install_Guide/Final_Preparations_and_Reboot_into_EFI  
Configuring systemd: https://wiki.gentoo.org/wiki/User:Sakaki/Sakaki%27s_EFI_Install_Guide/Configuring_systemd_and_Installing_Necessary_Tools  
KDE on Gentoo: https://wiki.gentoo.org/wiki/KDE
