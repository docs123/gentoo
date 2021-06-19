Follow the guide: https://wiki.gentoo.org/wiki/Handbook:AMD64

## Preparing the disks

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks

GPT + UEFI, we want to keep it as close to defaults as possible

    sudo fdisk /dev/sda
    sudo mkfs.vfat -F 32 /dev/sda1
    sudo mkfs.ext4 /dev/sda2

Mount the partitions

    sudo mkdir /mnt/gentoo
    sudo mount /dev/sda2 /mnt/gentoo

## Stage 3

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage

    cd /mnt/gentoo
    sudo wget https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20210616T214502Z/stage3-amd64-systemd-20210616T214502Z.tar.xz
    sudo tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
    sudo nano -w etc/portage/make.conf

Just add `-march=native` to `COMMON_FLAGS` and we're good to go
And set `-j2` for `MAKEOPTS`

# Installing the Gentoo base system

Section: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base

    sudo mkdir --parents etc/portage/repos.conf
    sudo cp usr/share/portage/config/repos.conf etc/portage/repos.conf/gentoo.conf
    sudo cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

mount

    sudo mount --types proc /proc /mnt/gentoo/proc
    sudo mount --rbind /sys /mnt/gentoo/sys
    sudo mount --make-rslave /mnt/gentoo/sys
    sudo mount --rbind /dev /mnt/gentoo/dev
    sudo mount --make-rslave /mnt/gentoo/dev

chroot

    sudo chroot /mnt/gentoo /bin/bash
    source /etc/profile
    export PS1="(chroot) ${PS1}"

Now time to work

    mount /dev/sda1 /boot
    emrerge-webrsync
    eselect news list
    eselect news read
    eselect profile list
    eselect profile set 9

Set the desktop/plasma/systemd profile

    emerge --ask --verbose --update --deep --newuse @world

And go to sleep...
