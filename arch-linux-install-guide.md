# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

A from-scratch Arch Linux install for a UEFI machine with a single disk. Follow the steps in
order. At a **choice** step, pick one option and follow its link. An **optional** step links to a
short extra doc you can take or skip. Every linked doc ends by sending you to the next step.

**Placeholders:** a value in angle brackets, like `/dev/<your-disk>` or `<your-username>`, must
be replaced with your own. Plain values like `wlan0` are examples or names this guide creates -
check the relevant command's output for yours.

## Pre-Installation

### 1.0 Set Keyboard Layout
```shell
loadkeys us
```
Sets the live session's keyboard layout, so everything you type from here on (commands and
passwords) comes out as expected. US is the default. Common alternatives: `uk` (British), `ca`
(Canadian), `de` (German), `dvorak`, `dvorak-programmer`, `dvorak-l`/`dvorak-r` (one-handed
Dvorak), `colemak`. `localectl list-keymaps` lists every option; pass yours to `loadkeys` in
place of `us`.

### 2.0 Verify UEFI Boot Mode
```shell
ls /sys/firmware/efi/efivars
```
This directory only exists when booted in UEFI mode. If it's missing, you're in legacy BIOS mode
and this guide doesn't apply.

### 3.0 Connect to the Internet
A wired connection configures itself. Check it:
```shell
ping -c 3 archlinux.org
```
**Optional - on Wi-Fi?** -> [Connect to Wi-Fi](options/03-wifi.md)

### 4.0 List Disks
```shell
lsblk
```
Find your target disk, e.g. `/dev/nvme0n1` (NVMe) or `/dev/sda` (SATA/virtio). The rest of this
guide calls it `/dev/<your-disk>`. Double-check it: the next steps erase it.

## Disk Setup

### 5.0 Partition the Disk
```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
Create a small EFI System partition for the bootloader and one Linux partition for everything
else:

```shell
# delete existing partition(s) to make room for your new partition scheme
select [ Delete ]

# Set up the EFI system partition
select [ New ]

Partition Size: 1G

select [ Type ] "EFI System"

# Set up the Linux partition
select [ New ]

Partition Size: accept default value (uses the remaining free space)

select [ Write ]
# example cfdisk output - your sizes and disk name will differ
|Number | Start (sector) | End (sector) | Size   | Code | Name             |
|------ | -------------- | ------------ | ------ | ---- | ---------------- |
|1      | 2048           | 1130495      | 1G     | EF00 | EFI System       |
|2      | 1130496        | 976773134    | 475.9G | 8300 | Linux Filesystem |
```
Run `lsblk` again for the new partition names. NVMe disks add a `p` before the number
(`/dev/nvme0n1p1`); SATA/virtio disks don't (`/dev/sda1`). The rest of this guide calls them
`/dev/<your-efi-partition>` and `/dev/<your-linux-partition>`.

### 6.0 Set Up the Disk
**Choice - pick one.** This decides how the Linux partition is used, and can't be changed later
without reinstalling.

- [Plain partitions (default)](options/06-disk-plain.md) - one ext4 root filesystem and a
  swapfile. Simplest.
- [Encrypted (LUKS)](options/06-disk-luks.md) - the disk is unreadable without your passphrase if
  the machine is lost or stolen.
- [LVM](options/06-disk-lvm.md) - separate resizable volumes for root, `/var`, `/tmp`, swap, and
  `/home`.
- [LVM inside LUKS](options/06-disk-lvm-on-luks.md) - the LVM volumes inside one encrypted
  partition.

## Base Installation

### 7.0 Install Essential Packages
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio lvm2 bash-completion dhcpcd iwd nano
```
Installs the base system, kernel, firmware, initramfs builder, LVM tools, shell completions,
networking (`dhcpcd`, `iwd`), and the `nano` text editor, which this guide's editor steps use; any
editor works. `cryptsetup`, for encrypted disks, comes with `base`; `lvm2` is harmless on a disk
without LVM. Add `neovim` or `vim` to the list if you want one of them as well.

### 8.0 Generate fstab
```shell
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```
Writes every current mount and active swap under `/mnt` into the new system's `/etc/fstab`, keyed
by UUID (`-U`) since device names can change between boots. It should list `/`, `/boot`, any
other volumes, and swap.

### 9.0 Chroot into New System
```shell
arch-chroot /mnt
```
Every command from here until 18.0 runs inside the new system instead of the live ISO.

## Configure the System

### 10.0 Set Time, Locale, and Keymap
```shell
ln -sf /usr/share/zoneinfo/UTC /etc/localtime
hwclock --systohc
systemctl enable systemd-timesyncd
```
Sets the timezone, writes the time to the hardware clock, and enables automatic clock sync. `UTC`
avoids daylight-saving changes; for local time, use your zone instead, e.g.
`/usr/share/zoneinfo/America/New_York` (`ls /usr/share/zoneinfo` lists them).

#### Uncomment your locale(s) in /etc/locale.gen:
```shell
nano /etc/locale.gen
```
The default is `en_US.UTF-8 UTF-8`. Alternatives include `en_GB.UTF-8 UTF-8` and
`en_CA.UTF-8 UTF-8`.

#### Generate the locale:
```shell
locale-gen
```

#### Create /etc/locale.conf:
```shell
nano /etc/locale.conf
```
```conf
# /etc/locale.conf
LANG=en_US.UTF-8
```
Replace `en_US.UTF-8` with the locale you uncommented.

#### Create /etc/vconsole.conf:
```shell
nano /etc/vconsole.conf
```
```conf
# /etc/vconsole.conf
KEYMAP=us
```
`KEYMAP` is your keyboard layout on the console at every boot, including an encrypted disk's
passphrase prompt: `us` for a US keyboard, otherwise e.g. `uk`, `ca`, `de`, `dvorak`,
`dvorak-programmer`, `dvorak-l`/`dvorak-r`, or `colemak`.

### 11.0 Build the Initramfs
```shell
mkinitcpio -P
```
The initramfs is a small root filesystem the kernel loads first to prepare for mounting your real
root. This rebuilds it with your console keymap and the disk settings already saved in
`/etc/mkinitcpio.conf.d/disk.conf`.

## Boot Loader

### 12.0 Install the Bootloader
**Choice - pick one.**

- [systemd-boot (default)](options/12-systemd-boot.md) - minimal, and part of systemd.
- [Limine](options/12-limine.md) - a small standalone bootloader.

## Accounts and Networking

### 13.0 Set Hostname
#### Create /etc/hostname:
```shell
nano /etc/hostname
```
```conf
<your-hostname>
```
e.g. `desktop`.

#### Add your hostname to /etc/hosts:
```shell
nano /etc/hosts
```
Add these lines:
```conf
127.0.0.1   localhost
::1         localhost
127.0.1.1   <your-hostname>.localdomain   <your-hostname>  # e.g. desktop.localdomain desktop
```
So your hostname resolves without a network.

### 14.0 Enable Networking Services
```shell
systemctl enable dhcpcd
systemctl enable iwd.service
```
Starts networking on every boot: `iwd` for Wi-Fi, `dhcpcd` for addresses.

### 15.0 Set Root Password
```shell
passwd
```

### 16.0 Add User
```shell
useradd -m -G wheel -s /bin/bash <your-username>  # e.g. archie
passwd <your-username>                             # e.g. archie
```
Creates your everyday user with a home directory (`-m`), in the `wheel` group (`-G wheel`), with
bash as its login shell (`-s /bin/bash`).

### 17.0 Configure Privilege Escalation
**Choice - pick one.** This lets your user run commands as root.

- [sudo (default)](options/17-sudo.md) - the most widely used tool.
- [doas](options/17-doas.md) - a much smaller, simpler alternative.

## Finalize and Reboot

### 18.0 Exit Chroot
```shell
exit
```

### 19.0 Unmount All Partitions
```shell
umount -l /mnt
```

### 20.0 Reboot into the New System
```shell
reboot
```
Remove the installation media so the machine boots from disk.

### 21.0 Log In
Log in as your user. A wired connection comes up by itself.

**Optional - on Wi-Fi?** -> [Connect to Wi-Fi](options/21-wifi.md)

### 22.0 Verify Installation
```shell
ping -c 3 archlinux.org        # Confirm network
lsblk                          # Confirm partition layout
swapon --show                  # Verify swap active
cat /etc/fstab                 # Sanity-check mount entries
```
`lsblk` should show your root filesystem mounted at `/` and the EFI partition at `/boot`,
`swapon --show` should list an active swap, and fstab should have one line per mounted filesystem
plus one for swap. Reaching this login prompt already proves the bootloader works.

## Post-Install Configuration

Your system works as it is. Every step below is optional; skip any you don't want. Run them as
your regular user.

### 23.0 CPU Microcode
**Optional:** microcode updates fix CPU bugs and security issues. `lscpu` shows your CPU vendor.
-> [AMD](options/23-microcode-amd.md) or [Intel](options/23-microcode-intel.md)

### 24.0 Graphics Drivers
**Optional:** open-source drivers for Intel and AMD GPUs. ->
[Install mesa](options/24-mesa.md)

### 25.0 32-bit Packages
**Optional:** the `multilib` repository holds 32-bit packages that Steam and some games need. ->
[Enable multilib](options/25-multilib.md)

### 26.0 AUR Build Tools
**Optional:** the tools to build packages from the Arch User Repository. ->
[Install the AUR build tools](options/26-aur-tools.md)

### 27.0 Firewall
**Optional:** block every unsolicited inbound connection. -> [Set up ufw](options/27-ufw.md)

### 28.0 SSH Server
**Optional:** log in from other machines, with key-only login and brute-force protection. ->
[Set up OpenSSH](options/28-ssh-server.md)

### 29.0 Update Hygiene
Some Arch upgrades need manual steps, announced on
[archlinux.org/news](https://archlinux.org/news/) before they land. Update regularly, after
reading the news for anything posted since your last upgrade:
```shell
sudo pacman -Syu
```
**Optional:** a daily list of pending updates, so you know when to check. ->
[Set up a daily update reminder](options/29-update-reminder.md)

### 30.0 Network Manager
**Optional:** replace `iwd`/`dhcpcd` with NetworkManager, which desktop environments' network
widgets expect. -> [Switch to NetworkManager](options/30-networkmanager.md)

### 31.0 Power Management
**Optional, mainly for laptops:** ->
[power-profiles-daemon](options/31-power-profiles-daemon.md) (simple profiles, switched from a
desktop's battery widget) or [TLP](options/31-tlp.md) (thorough automatic tuning)

### 32.0 Time Sync
**Optional:** replace `systemd-timesyncd` with `chrony`, for finer control over time sync. ->
[Switch to chrony](options/32-chrony.md)

### 33.0 Shell
**Optional:** -> [Customize bash](options/33-bash-config.md), or switch your login shell to
[zsh](options/33-zsh.md) or [fish](options/33-fish.md)

### 34.0 Dotfiles
**Optional:** keep your config files in version control. ->
[Manage dotfiles](options/34-dotfiles.md)

## You're Done

You now have a bootable Arch system configured the way you chose.
