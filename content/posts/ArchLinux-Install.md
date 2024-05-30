+++
author = 'elez_wq'
title = "Привет Мир"
date = "2024-05-24T00:12:49+03:00"
description = ""
draft = false
+++
1 Markup disk efi

    1.1 Erase disk

        mkfs.btrfs -f /dev/sda

    1.2 Markup disk efi

        cfdisk -z /dev/sda

        choose gpt

        create a boot partition /dev/sda1 -- 256M -- type -- EFI

        /sda2 -- 476G

    1.3 # Format disks

        create a system partition /dev

        mkfs.fat -F32 /dev/sda1

        mkfs.btrfs -f -L system /dev/sda2


2 Mount Disk

    2.1 # Mount btrfs

        mount /dev/sda2 /mnt

    2.2 # Btrfs create subvolumes

        btrfs su cr @

        btrfs su cr @home

        btrfs su cr @var

    2.3 Umount and mount btrfs on subvolumes

        umount /mnt

        mount -o rw,noatime,compress=zstd:3,ssd,ssd_spread,space_cache=v2,discard=async,subvol=@ /dev/sda2 /mnt

        mount -o rw,noatime,compress=zstd:3,ssd,ssd_spread,space_cache=v2,discard=async,subvol=@home /dev/sda2 /mnt/home

        mount -o rw,noatime,compress=zstd:3,ssd,ssd_spread,space_cache=v2,discard=async,subvol=@var /dev/sda2 /mnt/var

        mount /dev/sda1 /mnt


3 Install base system

    3.1 Install base system

        pacstrap -i /mnt base base-devel linux-zen linux-zen-headers linux-firmware amd-ucode iucode-tool btrfs-progs nano

    3.2 Genfstab

        genfstab -pU /mnt >> /mnt/etc/fstab


4 Configure base system


5 Install "systemd-boot"

    5.1 Install bootloader

        bootctl install

    5.2 Configure loader

        open conf file: nano /boot/loader/loader.conf

default linux.conf
timeout 0
console-mode auto
editor no

    5.3 Configure entries

        open conf file: nano /boot/loader/entries/linux.conf

tittle linux
linux  /vmlinuz-linux-zen
initrd /intel-ucode.img
initrd /initramfs-linux-zen.img
options root="LABEL=system" rw rootflags=subvol=@ nowatchdog rootfstype=btrfs


6 Microcode

    6.1 amd and intel microcode

        sudo pacman -S iucode-tool

    6.2 amd microcode

        sudo pacman -S amd-ucode

    6.3 intel microcode

        sudo pacmna -S intel-ucode


7.Tweaks

    7.1 mkinitcpio conf:

        open conf file: sudo nano /etc/mkinitcpio.conf

        edit "MODULES" to MODULES=(crc32c libcrc32c zlib_deflate btrfs)


    7.2 Swap (zram)

    install "zram-generator"

        sudo pacman -S zram-generator

    conf file zram-generator: sudo nano /etc/systemd/zram-generator.conf

[zram0]
zram-size = ram
scompression-algorithm = zstd


9.3 Drivers

    9.1 Amd drivers

        9.1.1 install driver

            sudo pacman -S mesa lib32-mesa mesa-vdpau lib32-mesa-vdpau vulkan-radeon lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver xf86-video-amdgpu

        9.1.2 Xorg configuration

            edit file, open file: sudo nano /etc/X11/xorg.conf.d/20-amdgpu.conf

            Section "OutputClass"
                 Identifier "AMD"
                 MatchDriver "amdgpu"
                 Driver "amdgpu"
            EndSection