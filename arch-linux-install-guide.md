# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

A from-scratch Arch Linux install for a UEFI machine with a single disk, as one sequence of
steps. Some steps have a single set of commands. Others list options, one marked **(default)**:
pick the option that fits, run only its commands, then go on to the next step.

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

#### Option: Wired (default)
DHCP configures itself; nothing to run.

#### Option: Wi-Fi
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

### 6.0 Set Up Encryption and LVM
Choose how the Linux partition is used. This can't be changed later without reinstalling. The
EFI partition always stays unencrypted, because UEFI firmware must read it.

#### Option: Plain partitions (default)
One ext4 root filesystem directly on the Linux partition. Nothing to run.

#### Option: Encrypted disk (LUKS)
The disk is unreadable without your passphrase if the machine is lost, stolen, or accessed while
powered off.
```shell
cryptsetup luksFormat /dev/<your-linux-partition>      # e.g. /dev/nvme0n1p2
cryptsetup open /dev/<your-linux-partition> cryptroot  # e.g. /dev/nvme0n1p2
```
Type `YES` to confirm, then set a passphrase. **There is no recovery if you forget it**, so store
it somewhere safe, such as a password manager. `open` unlocks the partition as
`/dev/mapper/cryptroot`.

#### Option: LVM
Separate volumes for root, `/var`, `/tmp`, swap, and `/home`, so a runaway `/var` can't fill
root. Volumes can be resized later.
```shell
pvcreate /dev/<your-linux-partition>     # e.g. /dev/nvme0n1p2
vgcreate vg /dev/<your-linux-partition>  # e.g. /dev/nvme0n1p2
lvcreate -L 20G vg -n root               # OS and packages
lvcreate -L 20G vg -n var                # logs, caches, databases
lvcreate -L 8G vg -n tmp                 # temporary files
lvcreate -L 4G vg -n swap                # adjust to match RAM if hibernating
lvcreate -l 100%FREE vg -n home          # remaining space
```
Creates a volume group named `vg` and carves the volumes out of it. Adjust the sizes to your
drive; on a 256G drive, for example, 10G `/var` and 4G `/tmp` leave more for `/home`.

#### Option: LVM inside LUKS
The LVM volumes, inside one encrypted partition: one passphrase unlocks them all.
```shell
cryptsetup luksFormat /dev/<your-linux-partition>      # e.g. /dev/nvme0n1p2
cryptsetup open /dev/<your-linux-partition> cryptroot  # e.g. /dev/nvme0n1p2
pvcreate /dev/mapper/cryptroot
vgcreate vg /dev/mapper/cryptroot
lvcreate -L 20G vg -n root               # OS and packages
lvcreate -L 20G vg -n var                # logs, caches, databases
lvcreate -L 8G vg -n tmp                 # temporary files
lvcreate -L 4G vg -n swap                # adjust to match RAM if hibernating
lvcreate -l 100%FREE vg -n home          # remaining space
```
Type `YES` to confirm, then set a passphrase. **There is no recovery if you forget it**, so store
it somewhere safe. Adjust the volume sizes to your drive; on a 256G drive, for example, 10G
`/var` and 4G `/tmp` leave more for `/home`.

### 7.0 Format the Partitions

#### Option: Plain partitions (default)
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/<your-linux-partition>    # e.g. /dev/nvme0n1p2
```

#### Option: Encrypted disk (LUKS)
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/mapper/cryptroot
```

#### Option: LVM, or LVM inside LUKS
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 -L "Arch Root" /dev/vg/root
mkfs.ext4 -L "Arch Var"  /dev/vg/var
mkfs.ext4 -L "Arch Tmp"  /dev/vg/tmp
mkfs.ext4 -L "Arch Home" /dev/vg/home
mkswap /dev/vg/swap
```

UEFI requires FAT32 on the EFI partition; everything else gets ext4.

### 8.0 Mount the Partitions

#### Option: Plain partitions (default)
```shell
mount /dev/<your-linux-partition> /mnt     # e.g. /dev/nvme0n1p2
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```

#### Option: Encrypted disk (LUKS)
```shell
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```

#### Option: LVM, or LVM inside LUKS
```shell
mount /dev/vg/root /mnt
mkdir -p /mnt/{home,var,tmp,boot}
mount /dev/vg/home /mnt/home
mount /dev/vg/var  /mnt/var
mount /dev/vg/tmp  /mnt/tmp
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```

The new system gets installed under `/mnt` in the next step.

## Base Installation

### 9.0 Install Essential Packages

#### Option: Plain partitions or encrypted disk (default)
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd nano
```

#### Option: LVM, or LVM inside LUKS
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd nano lvm2
```
`lvm2` lets the installed system find its volumes at boot.

Installs the base system, kernel, firmware, initramfs builder, shell completions, networking
(`dhcpcd`, `iwd`), and the `nano` text editor, which this guide's commands use. `cryptsetup`, for
encrypted disks, comes with `base`. Add `neovim` or `vim` to the list if you want one of them as
well.

### 10.0 Generate fstab
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
Writes the mounts under `/mnt` into the new system's `/etc/fstab`, keyed by UUID (`-U`) since
device names can change between boots.

#### Option: Leave /tmp as is (default)
Nothing to run.

#### Option: Harden /tmp (LVM, or LVM inside LUKS)
`/tmp` has its own volume, so it can get stricter mount options. Open the new fstab:
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

## Configure the System

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
console at every boot, including an encrypted disk's passphrase prompt: `us` for a US keyboard,
otherwise e.g. `uk`, `ca`, `de`, `dvorak`, `dvorak-programmer`, `dvorak-l`/`dvorak-r`, or
`colemak`.

### 13.0 Configure Swap

#### Option: Swapfile - plain partitions or encrypted disk (default)
```shell
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```
Creates and activates a 4G swapfile (`count=4096` MiB; raise it to roughly your RAM size if you
want hibernation). `genfstab` ran before the file existed, so the last line adds it to fstab. On
an encrypted disk, the swapfile is encrypted along with root.

#### Option: Swap volume - LVM, or LVM inside LUKS
```shell
swapon /dev/vg/swap
echo '/dev/vg/swap none swap defaults 0 0' >> /etc/fstab
```
Activates the `swap` logical volume and adds it to fstab.

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
Set `MODULES` to load FAT32 (the EFI partition's filesystem) early at boot:
```conf
MODULES=(vfat)
```
Then replace the `HOOKS` line with one of the options below.

#### Option: Plain partitions (default)
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```

#### Option: Encrypted disk (LUKS)
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck)
```

#### Option: LVM
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block lvm2 filesystems fsck)
```

#### Option: LVM inside LUKS
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```

Hooks run in order. `encrypt` unlocks the disk and `lvm2` finds the volumes, so both come after
`block` and before `filesystems`, with `lvm2` after `encrypt`. These lines use `udev`, `keymap`,
and `consolefont` in place of the stock `systemd` and `sd-vconsole` hooks; `keymap` copies the
`KEYMAP` from `/etc/vconsole.conf` into the initramfs, so your layout applies at the passphrase
prompt. `microcode` embeds CPU microcode updates, if installed.

#### Rebuild initramfs:
```shell
mkinitcpio -P
```

### 15.0 Install the Bootloader

#### Set your kernel options
Run the line for your disk. It stores the kernel options in `$OPTS` for the bootloader commands
below, so run those in this same shell.

**Option: Plain partitions (default)**
```shell
OPTS="root=UUID=$(blkid -s UUID -o value /dev/<your-linux-partition>) rw"
```

**Option: Encrypted disk (LUKS)**
```shell
OPTS="cryptdevice=UUID=$(blkid -s UUID -o value /dev/<your-linux-partition>):cryptroot root=/dev/mapper/cryptroot rw"
```

**Option: LVM**
```shell
OPTS="root=/dev/vg/root rw"
```

**Option: LVM inside LUKS**
```shell
OPTS="cryptdevice=UUID=$(blkid -s UUID -o value /dev/<your-linux-partition>):cryptroot root=/dev/vg/root rw"
```

`root=` points at your root filesystem. `cryptdevice=` tells the `encrypt` hook which partition
to unlock (by UUID, since device names can change) and to name it `cryptroot`. Check the result:
```shell
echo "$OPTS"
```

#### Option: systemd-boot (default)
```shell
bootctl install
cat > /boot/loader/loader.conf <<EOF
default arch.conf
timeout 3
console-mode max
editor no
EOF
cat > /boot/loader/entries/arch.conf <<EOF
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options $OPTS
EOF
cat > /boot/loader/entries/arch-fallback.conf <<EOF
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
options $OPTS
EOF
```
`bootctl install` copies systemd-boot onto the EFI partition and registers it with the UEFI
firmware. `loader.conf` boots `arch.conf` after 3 seconds, uses the highest text resolution, and
disables editing kernel parameters from the boot menu, so someone at the keyboard can't change
them. The fallback entry boots an initramfs with more drivers, as a recovery option.

#### Option: Limine
```shell
pacman -S limine
mkdir -p /boot/EFI/BOOT
cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/
cat > /boot/limine.conf <<EOF
timeout: 5

/Arch Linux (linux)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux.img
    cmdline: $OPTS rootfstype=ext4 add_efi_memmap vsyscall=none

/Arch Linux (linux-fallback)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux-fallback.img
    cmdline: $OPTS rootfstype=ext4 add_efi_memmap vsyscall=none
EOF
```
Copies Limine to the EFI partition's fallback boot path, which most UEFI firmware boots
automatically without a registered boot entry, and writes its menu. The fallback entry boots an
initramfs with more drivers, as a recovery option.

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

### 17.0 Enable Networking Services

#### Option: iwd and dhcpcd (default)
```shell
systemctl enable dhcpcd
systemctl enable iwd.service
```
Lightweight command-line networking: `iwd` for Wi-Fi, `dhcpcd` for addresses.

#### Option: NetworkManager
```shell
pacman -S networkmanager
systemctl enable NetworkManager
```
One service for wired and Wi-Fi that remembers networks, and what desktop environments' network
widgets (GNOME, KDE, XFCE) expect.

### 18.0 Set Root Password
```shell
passwd
```

### 19.0 Add User
```shell
useradd -m -G wheel -s /bin/bash <your-username>  # e.g. archie
passwd <your-username>                             # e.g. archie
```
Creates your everyday user with a home directory (`-m`), in the `wheel` group (`-G wheel`), with
bash as its login shell (`-s /bin/bash`).

### 20.0 Configure Privilege Escalation

#### Option: sudo (default)
```shell
pacman -S sudo
EDITOR=nano visudo
```
`visudo` edits `/etc/sudoers` and checks its syntax before saving, since a broken file can lock
you out of root. Uncomment this line, then save and exit:
```conf
%wheel ALL=(ALL:ALL) ALL
```
Members of `wheel`, including your user, can now run any command as root with `sudo`.

#### Option: doas
`opendoas` is a port of OpenBSD's `doas`: a much smaller codebase and simpler config than `sudo`,
but less common and less configurable.
```shell
pacman -S opendoas
echo "permit persist <your-username>" > /etc/doas.conf  # e.g. archie
chmod 600 /etc/doas.conf
ln -s /usr/bin/doas /usr/local/bin/sudo
```
Lets your user run any command as root with `doas <command>`. `persist` skips the password prompt
for five minutes after you enter it, and `chmod 600` makes the file readable only by root. The
link makes `sudo <command>` run `doas <command>`, so the `sudo` commands in the rest of this
guide, your muscle memory, and simple scripts all work unchanged; `sudo`-only options, such as
`sudo -E`, won't.

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

### 24.0 Log In and Connect to the Network
Log in as your user.

#### Option: Wired (default)
Already connected; nothing to run.

#### Option: Wi-Fi with iwd
```shell
iwctl station wlan0 connect <your-ssid>  # e.g. connect MyHomeWiFi
```
Replace `wlan0` with your interface (`iwctl device list` shows it). `iwd` remembers the network
from now on.

#### Option: Wi-Fi with NetworkManager
```shell
nmcli device wifi connect <your-ssid> password <your-wifi-password>
```
NetworkManager remembers the network from now on.

#### Test connectivity:
```shell
ping -c 3 archlinux.org
```

### 25.0 Verify Installation
```shell
lsblk                          # Confirm partition layout
swapon --show                  # Verify swap active
cat /etc/fstab                 # Sanity-check mount entries
```
`lsblk` should show your root filesystem mounted at `/` and the EFI partition at `/boot`,
`swapon --show` should list an active swap, and fstab should have one line per mounted filesystem
plus one for swap. Reaching this login prompt already proves the bootloader works.

## Post-Install Configuration

Your system works as it is. Every remaining step is optional and defaults to leaving things as
they are. Run them as your regular user.

### 26.0 CPU Microcode
Microcode updates fix CPU bugs and security issues. `lscpu` shows your CPU vendor.

#### Option: Skip (default)

#### Option: AMD
```shell
sudo pacman -S amd-ucode
sudo mkinitcpio -P
```

#### Option: Intel
```shell
sudo pacman -S intel-ucode
sudo mkinitcpio -P
```

Rebuilding the initramfs embeds the update, so it loads early at boot with no bootloader changes.

### 27.0 Graphics Drivers

#### Option: Skip (default)

#### Option: Intel or AMD GPU
```shell
sudo pacman -S mesa
```
`mesa` provides the open-source OpenGL/Vulkan drivers for Intel and AMD GPUs. Nvidia GPUs need
one of the `nvidia*` packages instead, which this guide doesn't cover.

### 28.0 32-bit Packages (multilib)

#### Option: Skip (default)

#### Option: Enable multilib
The `multilib` repository holds 32-bit packages that Steam and some games need.
```shell
sudo nano /etc/pacman.conf
```
Uncomment this section, then refresh the package databases and upgrade:
```conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```
```shell
sudo pacman -Syu
```

### 29.0 AUR Build Tools

#### Option: Skip (default)

#### Option: Install them
```shell
sudo pacman -S --needed binutils make gcc pkg-config fakeroot debugedit git
```
The AUR (Arch User Repository) ships build recipes (`PKGBUILD`s), not binaries, so installing
from it means compiling locally. This installs the build toolchain most `PKGBUILD`s expect, plus
`git` to fetch them.

### 30.0 Firewall

#### Option: No firewall (default)

#### Option: ufw
Blocks every unsolicited inbound connection while leaving outbound traffic alone.
```shell
sudo pacman -S ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo systemctl enable --now ufw
sudo ufw status verbose
```
Turns the firewall on now and on every boot. The status should show `Status: active`,
`deny (incoming)`, and `allow (outgoing)`. To make a service reachable later, allow its port:
`sudo ufw allow <port>/tcp` (e.g. `8080/tcp`).

### 31.0 SSH Server

#### Option: No SSH server (default)
Nobody can log in remotely.

#### Option: OpenSSH
-> [branches/ssh-server.md](branches/ssh-server.md) installs OpenSSH with key-only login, no
root login, and `fail2ban`.

### 32.0 Update Hygiene
Some Arch upgrades need manual steps, announced on
[archlinux.org/news](https://archlinux.org/news/) before they land.

#### Option: Update manually (default)
```shell
sudo pacman -Syu
```
Run this regularly, after reading the news for anything posted since your last upgrade.

#### Option: Daily update reminder
-> [branches/automatic-updates.md](branches/automatic-updates.md) sets up a daily check that
lists pending updates without installing them, and explains why fully automatic upgrades are
risky on Arch.

### 33.0 Power Management

#### Option: None (default)

#### Option: power-profiles-daemon
Performance/Balanced/Power Saver profiles you switch from a desktop environment's battery widget
(GNOME's talks to it directly). Simple, with few settings.
```shell
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon
```

#### Option: TLP
Tunes CPU scaling, USB autosuspend, disk power, and more automatically based on AC/battery state,
with many settings and no manual switching.
```shell
sudo pacman -S tlp
sudo systemctl enable --now tlp
```

### 34.0 Time Sync

#### Option: systemd-timesyncd (default)
Already running; nothing to do.

#### Option: chrony
Finer control: custom NTP servers, faster resync after suspend, and serving time to other
machines.
```shell
sudo pacman -S chrony
sudo systemctl disable --now systemd-timesyncd
sudo systemctl enable --now chronyd
chronyc tracking
```
Only one time-sync daemon should run, so this replaces `systemd-timesyncd` with `chronyd`.

### 35.0 Shell

#### Option: bash (default)
bash runs `~/.bashrc` every time it starts interactively. To customize it:
```shell
nano ~/.bashrc
```
```bash
alias ll='ls -lah'
export EDITOR=nano
PS1='[\u@\h \W]\$ '  # customize the prompt
```
Changes apply to new shells: open a new terminal, or run `source ~/.bashrc`.

#### Option: zsh
Bash-compatible, with a large ecosystem of frameworks such as Oh My Zsh.
```shell
sudo pacman -S zsh
chsh -s /usr/bin/zsh
```
`chsh` asks for your password; log out and back in to start using zsh. On its first launch, with
no `~/.zshrc` yet, zsh offers a setup wizard; follow it, or press `0` to create an empty
`~/.zshrc`. To customize it:
```shell
nano ~/.zshrc
```
```zsh
alias ll='ls -lah'
export EDITOR=nano
autoload -Uz compinit && compinit  # enable zsh's richer tab-completion
```
Changes apply to new shells: open a new terminal, or run `source ~/.zshrc`.

#### Option: fish
Not bash-compatible, but has autosuggestions, syntax highlighting, and tab-completion with no
configuration.
```shell
sudo pacman -S fish
chsh -s /usr/bin/fish
```
`chsh` asks for your password; log out and back in to start using fish. To customize it:
```shell
mkdir -p ~/.config/fish
nano ~/.config/fish/config.fish
```
```fish
alias ll 'ls -lah'
set -gx EDITOR nano
```
Changes apply to new shells: open a new terminal, or run `source ~/.config/fish/config.fish`.

### 36.0 Dotfiles

#### Option: Skip (default)

#### Option: Keep them in version control
Tracking your config files ("dotfiles": `.bashrc`, `.gitconfig`, `.vimrc`, and so on) lets you
restore them after a reinstall or copy them to a new machine. Two common approaches:

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
