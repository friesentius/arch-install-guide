# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

One path from a live Arch ISO to a configured system, for a UEFI machine with a single disk.
Each step shows a default inline. Where Arch offers a real alternative, the step links to a
branch in [`branches/`](branches/README.md), and every branch ends with a link back to the step
where you rejoin.

**Placeholders:** a value in angle brackets, like `/dev/<your-disk>` or `<your-username>`, must
be replaced with your own. Plain values like `wlan0` or `vg` are examples or names this guide
creates - check the relevant command's output for yours.

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
The default is one EFI partition, one ext4 root partition, and a swapfile (steps 6.0-8.0). Decide
now: switching layouts later means backing up and starting over from here.

**Want separate volumes for root/var/tmp/swap/home?** ->
[branches/lvm-disk-layout.md](branches/lvm-disk-layout.md) uses LVM, so a runaway `/var` can't
fill root, with a swap volume instead of a swapfile. Replaces 6.0-8.0.

**Want the disk encrypted?** -> [branches/disk-encryption.md](branches/disk-encryption.md) uses
LUKS, so the disk is unreadable without your passphrase if the machine is lost or stolen.
Replaces 6.0-8.0, and also covers combining encryption with LVM.

Both branches rejoin at 9.0. Later steps that differ by layout (9.0, 10.0, 14.0, 15.0, 20.0)
list every variant and show how to check which one is yours.

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
Check your disk layout with `lsblk -f`. `LVM2_member` anywhere in the output means LVM (with or
without encryption), which needs the extra `lvm2` package. Anything else (plain `ext4`, or
`crypto_LUKS` alone) uses the plain list.

**Plain partition layout, or encryption without LVM:**
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd openssh nano
```

**LVM (with or without encryption):**
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd openssh nano lvm2
```

This installs the base system, kernel, firmware, initramfs builder, shell completions, networking
(`dhcpcd`, `iwd`), SSH (`openssh` - remove it if you won't use it), and a text editor. The editor
defaults to `nano`; swap in `neovim` or `vim` if you prefer, and use it wherever this guide says
`nano`.

## Configure the System

### 10.0 Generate fstab
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
Writes the mounts under `/mnt` into the new system's `/etc/fstab`, keyed by UUID (`-U`) since
device names can change between boots.

**Optional, LVM only - harden `/tmp`.** Check with `lsblk -f`: this applies only if you see
`LVM2_member`, because only the LVM layout gives `/tmp` its own volume. Open the new fstab:
```shell
nano /mnt/etc/fstab
```
On the `/tmp` line `genfstab` wrote, keep the UUID and change the options to:
```shell
UUID=<your-tmp-volume-uuid>    /tmp    ext4    rw,noatime,nosuid,nodev    0 2
```
`nosuid` and `nodev` stop setuid binaries and device files from working on world-writable
`/tmp`. To also empty `/tmp` at every boot, and clear files older than a day (`1d`) in between:
```shell
echo "D /tmp 1777 root root 1d" > /mnt/etc/tmpfiles.d/clean-tmp.conf
```

### 11.0 Chroot into New System
```shell
arch-chroot /mnt
```
Every command from here until 21.0 runs inside the new system instead of the live ISO.

### 12.0 Set Time and Locale
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
Replace `en_US.UTF-8` with the locale you uncommented. For `KEYMAP`, test the layout you're
typing with right now: type `@` (Shift+2) and `:` (Shift+;). If both appear, keep `us`;
otherwise use your layout's `loadkeys` name (e.g. `dvorak`). This keymap applies to the console
on every boot and, through the `keymap` hook in 15.0, to early-boot prompts such as a LUKS
passphrase.

### 13.0 Network Configuration
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

### 14.0 Configure Swap
Check your disk layout with `lsblk -f`: `LVM2_member` anywhere means LVM, so use the swap
logical volume. Anything else (plain `ext4`, or `crypto_LUKS` alone) uses a swapfile.

**Swapfile (plain partition layout, or encryption without LVM):**
```shell
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```
Creates and activates a 4G swapfile (`count=4096` MiB; raise it to roughly your RAM size if you
want hibernation). `genfstab` ran before the file existed, so the last line adds it to fstab. On
an encrypted root, the swapfile is encrypted along with it.

**Swap logical volume (LVM, with or without encryption):**
```shell
swapon /dev/vg/swap
echo '/dev/vg/swap none swap defaults 0 0' >> /etc/fstab
```
Activates the `swap` volume created while partitioning and adds it to fstab.

#### Verify:
```shell
swapon --show
```

### 15.0 Initramfs Configuration
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
Replace the `HOOKS` line with the one matching your disk layout. Check with `lsblk -f`: look for
`crypto_LUKS` and/or `LVM2_member`.

**Plain partition layout (default):**
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```

**LVM only:**
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block lvm2 filesystems fsck)
```

**Encryption only:**
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck)
```

**LVM + encryption (LVM-on-LUKS):**
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```

Hooks run in order, so order matters. These lines use `udev`, `keymap`, and `consolefont` in
place of the stock `systemd` and `sd-vconsole` hooks. `keymap` copies the `KEYMAP` from
`/etc/vconsole.conf` into the initramfs so it applies at the LUKS passphrase prompt. `encrypt`
must come after `block`, and `lvm2` after `encrypt`, because LVM can only find the volume group
once the container is unlocked. `microcode` embeds CPU microcode updates, if installed.

#### Rebuild initramfs:
```shell
mkinitcpio -P
```

### 16.0 Enable Networking Services
```shell
systemctl enable dhcpcd
systemctl enable iwd.service
systemctl enable sshd        # Only if "pacman -Q openssh" shows it installed
```
Starts networking (and optionally SSH) automatically on every boot.

### 17.0 Set Root Password
```shell
passwd
```

### 18.0 Add User

**Want zsh or fish instead of bash?** -> [branches/alternate-shell.md](branches/alternate-shell.md),
run right after the commands below, then rejoin at 19.0.

```shell
useradd -m -G wheel -s /bin/bash <your-username>  # e.g. archie
passwd <your-username>                             # e.g. archie
```
Creates your everyday user with a home directory (`-m`), in the `wheel` group that 19.0 grants
root access (`-G wheel`), with bash as its login shell (`-s /bin/bash`).

### 19.0 Configure Privilege Escalation (sudo)

**Want doas instead?** `opendoas` is a smaller, simpler alternative ->
[branches/alternate-privilege-escalation.md](branches/alternate-privilege-escalation.md) replaces
this step, then rejoin at 20.0.

```shell
pacman -S sudo
EDITOR=nano visudo
```
`visudo` edits `/etc/sudoers` and checks its syntax before saving, since a broken file can lock
you out of root. Replace `nano` with your editor if `pacman -Q nano neovim vim 2>/dev/null`
shows a different one.

Uncomment this line, then save and exit:
```conf
%wheel ALL=(ALL:ALL) ALL
```
Members of `wheel`, including your user, can now run any command as root with `sudo`.

## Boot Loader

### 20.0 Install and Configure systemd-boot

**Want Limine instead?** -> [branches/limine-bootloader.md](branches/limine-bootloader.md)
replaces this step, then rejoin at 21.0.

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

#### Find your options line
Check your disk layout with `lsblk -f`: look for `crypto_LUKS` and/or `LVM2_member`. If you see
`crypto_LUKS`, get the UUID of the encrypted partition itself (not the `/dev/mapper` device):
```shell
blkid /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```

**Plain partition layout (default):**
```conf
options root=/dev/<your-root-partition> rw
```

**LVM only:**
```conf
options root=/dev/vg/root rw
```

**Encryption only:**
```conf
options cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/mapper/cryptroot rw
```
`cryptdevice=` tells the `encrypt` hook which partition to unlock (by the UUID `blkid` printed)
and to name it `cryptroot`; `root=` then points at the unlocked device.

**LVM + encryption (LVM-on-LUKS):**
```conf
options cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/vg/root rw
```
Same `cryptdevice=`, but `root=` points at the logical volume the `lvm2` hook finds inside the
unlocked container.

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
Replace the `options` line with yours from above.

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
ls /boot/limine.conf 2>/dev/null && echo "Limine" || bootctl status
```
Your partitions or volumes should be mounted where expected, swap (a swapfile or the `swap`
volume) should be active, and fstab should list root, the EFI partition, and swap with nothing
unexpected. The last line prints `Limine` if you're booting with Limine (reaching this login
prompt proves it works); otherwise `bootctl status` should report systemd-boot.

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

### 27.0 SSH Hardening
Default: skip. Check whether it applies with `systemctl is-enabled sshd 2>/dev/null`; if that
doesn't print `enabled`, there's no SSH server to harden.

**Want to lock it down?** -> [branches/ssh-hardening.md](branches/ssh-hardening.md) sets up
key-only login, disables root login, and adds `fail2ban`.

### 28.0 Update Hygiene
Default: run `pacman -Syu` regularly, prefixed with `sudo` or `doas` (check which you have with
`which sudo doas 2>/dev/null`), after reading [archlinux.org/news](https://archlinux.org/news/)
for anything needing manual steps.

**Want a scheduled reminder?** -> [branches/automatic-updates.md](branches/automatic-updates.md)
sets up a daily check that lists pending updates without installing them, and explains why fully
automatic upgrades are risky on Arch.

### 29.0 System Configuration
Default: keep `iwd`/`dhcpcd` networking, `systemd-timesyncd` time sync, and no power management.

**Want alternatives?** -> [branches/system-config.md](branches/system-config.md) covers
NetworkManager (for desktop network widgets), power management (`power-profiles-daemon` or TLP,
for laptops), and `chrony` time sync. Each is independent.

### 30.0 Shell Configuration and Dotfiles
Default: skip - your shell works with its stock config.

**Want to customize it?** -> [branches/shell-config.md](branches/shell-config.md) covers your
shell's config file and managing dotfiles long-term.

This is the last step.

## You're Done

Whichever branches you took, you now have a bootable Arch system configured the way you chose.

**Want encryption after all?** It can't be added to a running root filesystem: back up your data
and reinstall from 5.0 Choose Your Disk Layout.
