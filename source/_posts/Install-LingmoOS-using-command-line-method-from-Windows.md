---
title: Install LingmoOS using command line method from Windows
date: 2024-07-07 16:25:59
tags:
---
```bat
powershell start cmd -Verb runAs
mkdir %TEMP%\lingmo
cd %TEMP%\lingmo
set PATH=%PATH%;%TEMP%\lingmo

powershell Invoke-WebRequest -Uri "https://7-zip.org/a/7zr.exe" -OutFile "7zr.exe"
powershell Invoke-WebRequest -Uri "https://7-zip.org/a/7z2301-extra.7z" -OutFile "7z.7z"
powershell Invoke-WebRequest -Uri "https://github.com/aria2/aria2/releases/download/release-1.37.0/aria2-1.37.0-win-64bit-build1.zip" -OutFile "aria2.zip"
7zr e 7z.7z 7za.exe
7za e aria2.zip "aria2-*\aria2c.exe"
aria2c -x16 -j16 -s16 -k1M https://ftp.gnu.org/gnu/grub/grub-2.06-for-windows.zip -o grub.zip
aria2c -x16 -j16 -s16 -k1M https://jaist.dl.sourceforge.net/project/lingmo-os/release/iso/polaris/beta3.6/LingmoOS-2.0-Polaris-202404151043-Beta-desktop-amd64.iso -o lingmo.iso

diskpart
DISKPART> sel dis 0
DISKPART> sel vol=c
DISKPART> shr desired=20480
DISKPART> cre par pri size=4096
DISKPART> for fs=fat32 label="boot" quick
DISKPART> ass letter=D
DISKPART> cre par pri
DISKPART> for fs=ntfs label="lingmo" quick
DISKPART> exit

pushd D:\
7za x %temp%\lingmo\lingmo.iso
popd

bcdedit /enum | find "winload.exe"
set EFI=%ERRORLEVEL%
if %EFI% equ 1 (
    mountvol S: /S
    grub-install --target=x86_64-efi --efi-directory=S: --boot-directory=D:\boot
) else (
    grub-install --target=i386-pc --boot-directory=D:\boot \\.\PHYSICALDRIVE0
)
shutdown -r -t 0
```

```bash
sudo -i
apt update
apt install -y debootstrap
mkfs.ext4 /dev/disk/by-label/lingmo -L lingmo
mount /dev/disk/by-label/lingmo /mnt
debootstrap polaris /mnt https://packages.lingmo.org
cd /mnt
mount --bind /dev dev
mount --bind /proc proc
mount --bind /sys sys
mount --rbind /sys/firmware/efi/efivars sys/firmware/efi/efivars
mkdir -p boot/efi
mount /dev/disk/by-diskseq/1-part1 boot/efi
chroot .
apt install locales tzdata linux-image-amd64 grub-common grub-efi-amd64-bin grub-pc-bin
[ -e /sys/firmware/efi/fw_platform_size ]&&grub-install||grub-install /dev/disk/by-diskseq/1
echo "lingmo" > /etc/hostname
dpkg-reconfigure locales
dpkg-reconfigure tzdata
adduser lingmo
apt install xorg lingmo-*
systemctl enable sddm
exit
reboot
```
