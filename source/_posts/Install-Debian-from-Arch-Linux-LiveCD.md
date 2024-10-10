---
title: Install Debian from Arch Linux LiveCD
date: 2024-09-20 21:13:59
tags:
  - Tutorial
categories:
  - Linux
---
### *1* Install debootstrap
```shell
pacman -Sy debootstrap
```
### *2* Format partitions
Assuming that you already have this partition layout:
| Mount Point | File System | Device          |
|-------------|-------------|-----------------|
| `/boot/efi` | FAT32       | `/dev/nvme0n1p1`|
| `/`         | ext4        | `/dev/nvme0n1p2`|
| -           | swap        | `/dev/nvme0n1p3`|

```shell
mkfs.vfat -n EFI -F 32 /dev/nvme0n1p1
mkfs.ext4 -L debian /dev/nvme0n1p2
mkswap -L swap /dev/nvme0n1p3
```
### *3* Bootstrap the base system
```shell
mount /dev/disk/by-label/debian /mnt -o rw,noatime
debootstrap sid /mnt <package mirror, e.g. https://deb.debian.org/debian/>
```
### *4* Mount other partitions and generate fstab
```shell
mount --mkdir /dev/disk/by-label/EFI /mnt/boot/efi
swapon /dev/disk/by-label/swap
genfstab -L /mnt > /mnt/etc/fstab
```
### *5* Chroot stage
```shell
arch-chroot /mnt

# Fix PATH environment variable
export PATH=$PATH:/sbin:/usr/sbin

# Set hostname
echo debian > /etc/hostname

# Set root password
passwd

# Add a new user
adduser <username>

# Configure Locales
apt install locales
dpkg-reconfigure locales

# Configure timezone
dpkg-reconfigure tzdata

# Install kernel
apt install linux-image-amd64

# Install grub
apt install grub-efi
grub-install
grub-mkconfig -o /boot/grub/grub.cfg

# Install NetworkManager
apt install network-manager

# Exit chroot
exit
```
### *6* Run `reboot` and boot into the newly installed system
