# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

This is the core install guide: UEFI boot, a single EFI + ext4 root partition, a swapfile,
`pacstrap` base install, and **systemd-boot**. It gets you to a bootable, minimal Arch system.

For optional/specialized setups (LVM, the Limine bootloader, graphics drivers) that build on
or replace parts of this guide, see [`appendices/README.md`](appendices/README.md). See the
top-level [`README.md`](README.md) for an overview of the whole repo.

**A note on placeholders:** anywhere you see a value wrapped in angle brackets, like
`/dev/<your-disk>` or `<your-username>`, it is a placeholder you must replace with the real
value for your system. Anything shown as a plain, unbracketed value (like `wlan0` or `vg`) is
just an example or a name this guide invents along the way; check the relevant command's
output (`lsblk`, `iwctl device list`, etc.) for your actual value before continuing.

## Pre-Installation

### 1.0 Verify UEFI Boot Mode
```shell
ls /sys/firmware/efi/efivars
```
If the directory exists you're free to continue.

### 2.0 Set Keyboard Layout
```shell
localectl set-keymap us
```
Adjust keymap as needed (e.g., uk, de).

### 3.0 Connect to the Internet

#### Wired (DHCP):
```shell
ping -c 3 archlinux.org
```

#### Wi-Fi (Using IWD):
```shell
iwctl
device list                # Identify interface (e.g., wlan0)
station wlan0 scan         # Scan networks
station wlan0 get-networks # List networks
station wlan0 connect SSID # Replace SSID with your network name
exit
```

#### Test connectivity:
```shell
ping -c 3 archlinux.org
```

### 4.0 List Disks
```shell
lsblk
```
Identify your target disk (e.g., /dev/nvme0n1).

### 5.0 Partition the Disk
```shell
cfdisk /dev/nvme0n1
```

```shell
# delete existing partition to make room for your new partition scheme
select [ Delete ]

# Set up boot partition
select [ New ]

Partition Size: 1G

select [ Type ] "EFI System"
# Set up root partition
select [ New ]

Partition Size: accept default value

select [ Write ]
# cfdisk output
|Number | Start (sector) | End (sector) | Size   | Code | Name             |
|------ | -------------- | ------------ | ------ | ---- | ---------------- |
|1      | 2048           | 1130495      | 1G     | EF00 | EFI System       |
|2      | 1130496        | 976773134    | 475.9G | 8300 | Linux Filesystem |
```

### 6.0 Format the Partitions
```shell
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```

### 7.0 Mount the Partitions
```shell
mount /dev/nvme0n1p2 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```
/boot must remain unencrypted for UEFI boot.

## Base Installation

### Install Essential Packages
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd openssh nano
```
openssh (optional) remove unless you use ssh.

## Configure the System

### 1.0 Generate fstab
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

### 2.0 Chroot into New System
```shell
arch-chroot /mnt
```

### 3.0 Set Time and Locale
```shell
timedatectl set-ntp true
timedatectl set-timezone UTC # Avoids DST issues
hwclock --systohc --utc
```

#### Uncomment en_US.UTF-8 UTF-8 in /etc/locale.gen:
```shell
nano /etc/locale.gen
```

#### Generate and set locale:
```shell
locale-gen
localectl set-locale LANG="en_US.UTF-8"
localectl set-locale LC_TIME="en_US.UTF-8"
echo "KEYMAP=us" > /etc/vconsole.conf
```

### 4.0 Network Configuration
#### Set hostname:
```shell
echo myhostname > /etc/hostname
```

#### Edit /etc/hosts:
```shell
nano /etc/hosts
```

```shell
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain   myhostname
```

### 5.0 Configure Swap (swapfile)
```shell
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

#### Verify:
```shell
swapon --show
```

#### Add it to fstab:
```shell
echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```

### 6.0 Initramfs Configuration
#### Edit /etc/mkinitcpio.conf:
```shell
nano /etc/mkinitcpio.conf
```

#### Place vfat into MODULES:
```conf
MODULES=(vfat)
```

#### Replace the systemd hook with udev and sd-vconsole with consolefont (ordering matters):
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```

#### Rebuild initramfs:
```shell
mkinitcpio -P
```

### 7.0 Enable Networking Services
```shell
systemctl enable dhcpcd
systemctl enable iwd.service
systemctl enable sshd        # Enable if you installed openssh
```

### 8.0 Set Root Password
```shell
passwd
```

### 9.0 Add User
```shell
useradd -m -G wheel -s /bin/bash yourusername
passwd yourusername
```

### 10.0 Configure Privilege Escalation (opendoas)
```shell
pacman -S opendoas
```

#### Allow user to run commands as root:
```shell
echo "permit persist yourusername" > /etc/doas.conf
chmod 600 /etc/doas.conf
```
persist preserves password authentication for five minutes.

#### Optional: Add sudo alias:
```shell
echo "alias sudo=doas" >> /home/yourusername/.bashrc
```

### 11.0 Install and Configure systemd-boot
```shell
bootctl install
```

#### Edit /boot/loader/loader.conf:
```shell
nano /boot/loader/loader.conf
```

```conf
default arch.conf
timeout 3
console-mode max
editor no
```

#### Create /boot/loader/entries/arch.conf:
```shell
nano /boot/loader/entries/arch.conf
```

```conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=/dev/nvme0n1p2 rw
```

#### Optional: Create a fallback entry, /boot/loader/entries/arch-fallback.conf:
```shell
nano /boot/loader/entries/arch-fallback.conf
```

```conf
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
options root=/dev/nvme0n1p2 rw
```

## Finalize and Reboot
#### Exit chroot:
```shell
exit
```

#### Unmount all partitions:
```shell
umount -l /mnt
```

#### Reboot into the new system:
```shell
reboot
```

## Verify Installation
#### After logging in:
```shell
lsblk                          # Confirm partition layout
swapon --show                  # Verify swap active
bootctl status                 # Confirm systemd-boot is the active boot loader
cat /etc/fstab                 # Sanity-check mount entries
```
