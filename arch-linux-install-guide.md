# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

One path from a live Arch ISO to a configured system, for a UEFI machine with a single disk.
Each step shows a default inline. Where Arch offers a real alternative, the step links to a
branch in [`branches/`](branches/README.md). A branch contains every step that differs for its
choice, then sends you back here.

**Placeholders:** a value in angle brackets, like `/dev/<your-disk>` or `<your-username>`, must
be replaced with your own. Plain values like `wlan0` are examples or names this guide creates -
check the relevant command's output for yours.

## Pre-Installation

### 1.0 Set Keyboard Layout
```shell
loadkeys us
```
Sets the live session's keyboard layout, so everything you type from here on (commands and
passwords) comes out as expected. US is already the default. Common alternatives: `uk`
(British), `ca` (Canadian), `de` (German), `dvorak`, `dvorak-programmer`, `dvorak-l`/`dvorak-r`
(one-handed Dvorak), `colemak`. `localectl list-keymaps` lists every option; pass yours to
`loadkeys` in place of `us`.

### 2.0 Verify UEFI Boot Mode
```shell
ls /sys/firmware/efi/efivars
```
This directory only exists when booted in UEFI mode. If it's missing, you're in legacy BIOS mode
and this guide doesn't apply.

### 3.0 Connect to the Internet

#### Wired:
DHCP configures itself; skip to the test below.

#### Wi-Fi (iwd):
```shell
iwctl
device list                       # Identify interface (e.g., wlan0)
station wlan0 scan                # Scan networks
station wlan0 get-networks        # List networks
station wlan0 connect <your-ssid> # e.g. connect MyHomeWiFi
exit
```
Replace `wlan0` with the interface `device list` shows (e.g. `wlp3s0`), and `<your-ssid>` with
your network name.

#### Test connectivity:
```shell
ping -c 3 archlinux.org
```

### 4.0 List Disks
```shell
lsblk
```
Find your target disk, e.g. `/dev/nvme0n1` (NVMe) or `/dev/sda` (SATA/virtio). The rest of this
guide calls it `/dev/<your-disk>`. Double-check it: the next steps erase it.

## Disk Layout

### 5.0 Choose Your Disk Layout
The default is one EFI partition, one ext4 root partition, and a swapfile. Decide now: switching
layouts later means backing up and starting over from here.

**Want separate volumes for root/var/tmp/swap/home?** ->
[branches/lvm-disk-layout.md](branches/lvm-disk-layout.md) uses LVM, so a runaway `/var` can't
fill root, with a swap volume instead of a swapfile.

**Want the disk encrypted?** -> [branches/disk-encryption.md](branches/disk-encryption.md) uses
LUKS, so the disk is unreadable without your passphrase if the machine is lost or stolen.

**Want both?** -> [branches/lvm-on-luks.md](branches/lvm-on-luks.md) puts the LVM volumes inside
one encrypted LUKS container.

### 6.0 Partition the Disk
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

# Set up the root partition
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
`/dev/<your-efi-partition>` and `/dev/<your-root-partition>`.

### 7.0 Format the Partitions
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/<your-root-partition>     # e.g. /dev/nvme0n1p2
```
UEFI requires FAT32 on the EFI partition; root gets ext4.

### 8.0 Mount the Partitions
```shell
mount /dev/<your-root-partition> /mnt      # e.g. /dev/nvme0n1p2
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
The new system gets installed under `/mnt` in the next step.

## Base Installation

### 9.0 Install Essential Packages
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd nano
```
Installs the base system, kernel, firmware, initramfs builder, shell completions, networking
(`dhcpcd`, `iwd`), and the `nano` text editor, which this guide's commands use. Add `neovim` or
`vim` to the list if you want one of them as well.

## Configure the System

### 10.0 Generate fstab
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
Writes the mounts under `/mnt` into the new system's `/etc/fstab`, keyed by UUID (`-U`) since
device names can change between boots.

### 11.0 Chroot into New System
```shell
arch-chroot /mnt
```
Every command from here until 21.0 runs inside the new system instead of the live ISO.

### 12.0 Set Time, Locale, and Keymap
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

#### Generate and set locale and console keymap:
```shell
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "KEYMAP=us" > /etc/vconsole.conf
```
Replace `en_US.UTF-8` with the locale you uncommented. `KEYMAP` is your keyboard layout on the
console at every boot: `us` for a US keyboard, otherwise e.g. `uk`, `ca`, `de`, `dvorak`,
`dvorak-programmer`, `dvorak-l`/`dvorak-r`, or `colemak`.

### 13.0 Configure Swap
```shell
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```
Creates and activates a 4G swapfile (`count=4096` MiB; raise it to roughly your RAM size if you
want hibernation). `genfstab` ran before the file existed, so the last line adds it to fstab.

#### Verify:
```shell
swapon --show
```

### 14.0 Initramfs Configuration
The initramfs is a small root filesystem the kernel loads first to prepare for mounting your real
root. Edit its config:
```shell
nano /etc/mkinitcpio.conf
```

#### Set MODULES:
```conf
MODULES=(vfat)
```
Loads FAT32 (the EFI partition's filesystem) early at boot.

#### Set HOOKS:
Replace the `HOOKS` line with:
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```
Hooks run in order. This line uses `udev`, `keymap`, and `consolefont` in place of the stock
`systemd` and `sd-vconsole` hooks; `keymap` copies the `KEYMAP` from `/etc/vconsole.conf` into
the initramfs. `microcode` embeds CPU microcode updates, if installed.

#### Rebuild initramfs:
```shell
mkinitcpio -P
```

## Boot Loader

### 15.0 Install and Configure systemd-boot

**Want Limine instead?** -> [branches/limine-bootloader.md](branches/limine-bootloader.md)

```shell
bootctl install
```
Copies systemd-boot onto the EFI partition and registers it with the UEFI firmware.

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
Boots `arch.conf` (created below) after 3 seconds, uses the highest text resolution, and
disables editing kernel parameters from the boot menu, so someone at the keyboard can't change
them.

#### Create /boot/loader/entries/arch.conf:
```shell
nano /boot/loader/entries/arch.conf
```
```conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=/dev/<your-root-partition> rw
```
Replace `/dev/<your-root-partition>` with your root partition, e.g. `/dev/nvme0n1p2`.

#### Optional: Create a fallback entry, /boot/loader/entries/arch-fallback.conf:
```shell
nano /boot/loader/entries/arch-fallback.conf
```
```conf
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
options root=/dev/<your-root-partition> rw
```
Boots the fallback initramfs, which includes more drivers - a recovery option if an update breaks
normal boot. Use the same `options` line as `arch.conf`.

## Accounts and Networking

### 16.0 Network Configuration
#### Set hostname:
```shell
echo <your-hostname> > /etc/hostname  # e.g. echo desktop > /etc/hostname
```

#### Edit /etc/hosts:
```shell
nano /etc/hosts
```
Map your hostname to the loopback address, so it resolves without a network:

```shell
127.0.0.1   localhost
::1         localhost
127.0.1.1   <your-hostname>.localdomain   <your-hostname>  # e.g. desktop.localdomain desktop
```

Prefer NetworkManager over `iwd`/`dhcpcd`? That's a post-install swap at 29.0 System
Configuration.

### 17.0 Enable Networking Services
```shell
systemctl enable dhcpcd
systemctl enable iwd.service
```
Starts networking automatically on every boot.

### 18.0 Set Root Password
```shell
passwd
```

### 19.0 Add User
```shell
useradd -m -G wheel -s /bin/bash <your-username>  # e.g. archie
passwd <your-username>                             # e.g. archie
```
Creates your everyday user with a home directory (`-m`), in the `wheel` group that 20.0 grants
root access (`-G wheel`), with bash as its login shell (`-s /bin/bash`). Switching to zsh or fish
is a post-install step, at 30.0 Shell Configuration.

### 20.0 Configure Privilege Escalation (sudo)

**Want doas instead?** `opendoas` is a smaller, simpler alternative ->
[branches/alternate-privilege-escalation.md](branches/alternate-privilege-escalation.md)

```shell
pacman -S sudo
EDITOR=nano visudo
```
`visudo` edits `/etc/sudoers` and checks its syntax before saving, since a broken file can lock
you out of root.

Uncomment this line, then save and exit:
```conf
%wheel ALL=(ALL:ALL) ALL
```
Members of `wheel`, including your user, can now run any command as root with `sudo`.

## Finalize and Reboot

### 21.0 Exit Chroot
```shell
exit
```

### 22.0 Unmount All Partitions
```shell
umount -l /mnt
```

### 23.0 Reboot into the New System
```shell
reboot
```
Remove the installation media so the machine boots from disk.

## Verify Installation

### 24.0 Verify Installation
Log in as your user, then:
```shell
lsblk                          # Confirm partition layout
swapon --show                  # Verify swap active
cat /etc/fstab                 # Sanity-check mount entries
```
`lsblk` should show your root filesystem mounted at `/` and the EFI partition at `/boot`,
`swapon --show` should list an active swap, and fstab should have one line per mounted filesystem
plus one for swap. Reaching this login prompt already proves the bootloader works.

## Post-Install Configuration

Your system works as it is. Each remaining step is optional, defaults to skipping, and links to a
branch if you want more. Run them from your regular user.

### 25.0 Graphics, Microcode, and AUR Tools
Default: skip.

**Want them?** -> [branches/graphics-and-extras.md](branches/graphics-and-extras.md) covers CPU
microcode, graphics drivers with 32-bit support (e.g. for Steam), and the tools to build AUR
packages. Most desktop and gaming setups want these.

### 26.0 Firewall
Default: skip.

**Want one?** -> [branches/firewall.md](branches/firewall.md) sets up `ufw` to block all
unsolicited inbound connections.

### 27.0 SSH Server
Default: skip - with no SSH server, nobody can log in remotely.

**Want to log in from other machines?** -> [branches/ssh-server.md](branches/ssh-server.md)
installs OpenSSH with key-only login, no root login, and `fail2ban`.

### 28.0 Update Hygiene
Default: run `sudo pacman -Syu` regularly, after reading
[archlinux.org/news](https://archlinux.org/news/) for anything needing manual steps.

**Want a scheduled reminder?** -> [branches/automatic-updates.md](branches/automatic-updates.md)
sets up a daily check that lists pending updates without installing them, and explains why fully
automatic upgrades are risky on Arch.

### 29.0 System Configuration
Default: keep `iwd`/`dhcpcd` networking, `systemd-timesyncd` time sync, and no power management.

**Want alternatives?** -> [branches/system-config.md](branches/system-config.md) covers
NetworkManager (for desktop network widgets), power management (`power-profiles-daemon` or TLP,
for laptops), and `chrony` time sync. Each is independent.

### 30.0 Shell Configuration
Default: keep bash with its stock config.

**Want to customize bash?** -> [branches/shell-config.md](branches/shell-config.md) covers
aliases, environment variables, and the prompt in `~/.bashrc`.

**Want zsh or fish instead?** -> [branches/alternate-shell.md](branches/alternate-shell.md)
switches your login shell and sets up its config file.

### 31.0 Dotfiles
Default: skip.

Keeping your config files ("dotfiles": `.bashrc`, `.gitconfig`, `.vimrc`, and so on) in version
control lets you restore them after a reinstall or copy them to a new machine. Two common
approaches:

- **A bare git repository:** install git (`sudo pacman -S --needed git`), then
  `git init --bare ~/.dotfiles`, plus a `dotfiles` alias for
  `git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME`. You then `add`/`commit`/`push` files
  straight from `$HOME`, with no symlinks. Search "dotfiles bare git repository" for a full
  walkthrough. Simplest for one set of configs.
- **A dotfiles manager such as [chezmoi](https://www.chezmoi.io/):** templates handle differences
  between machines (hostnames, secrets, work vs. personal). More to learn, but pays off across
  several machines.

## You're Done

You now have a bootable Arch system configured the way you chose.
