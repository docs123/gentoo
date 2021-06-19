Follow the guide: https://wiki.gentoo.org/wiki/Handbook:AMD64

## Preparing the disks

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

## Stage 3

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage

```bash
cd /mnt/gentoo
sudo wget https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20210616T214502Z/stage3-amd64-systemd-20210616T214502Z.tar.xz
sudo tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
sudo nano -w etc/portage/make.conf
```

Just add `-march=native` to `COMMON_FLAGS` and we're good to go  
And set `-j2` for `MAKEOPTS`

## Installing the Gentoo base system

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




---

## References

Mastering Markdown: https://guides.github.com/features/mastering-markdown/
Gentoo cheat sheet: https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet

