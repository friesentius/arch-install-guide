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
This directory only exists if the system booted in UEFI mode rather than legacy BIOS. This
guide's later steps (the EFI partition, systemd-boot) only make sense on UEFI, so confirm this
first. If the directory doesn't exist, you're in BIOS/legacy mode and this guide doesn't apply
as written.

### 2.0 Set Keyboard Layout
```shell
localectl set-keymap us
```
Sets the keyboard layout for the live installer session, so keys you type (including passwords)
map to the characters you expect. This guide defaults to a US keymap. Common alternatives: `uk`
(British), `ca` (Canadian French/English), `de` (German). Run `localectl list-keymaps` to see
every keymap available.

### 3.0 Connect to the Internet
The rest of the install needs network access to download packages, so get online before
continuing.

#### Wired (DHCP):
```shell
ping -c 3 archlinux.org
```
If you're on a wired connection, DHCP usually configures itself automatically; this just
confirms you actually have connectivity.

#### Wi-Fi (Using IWD):
```shell
iwctl
device list                       # Identify interface (e.g., wlan0)
station wlan0 scan                # Scan networks
station wlan0 get-networks        # List networks
station wlan0 connect <your-ssid> # e.g. connect MyHomeWiFi
exit
```
`iwctl` is the interactive client for `iwd`, the Wi-Fi daemon included in the live ISO.
`wlan0` above is an example wireless interface name - yours may be different (e.g. `wlp3s0`);
use whatever `device list` actually shows you in every `station <interface> ...` command that
follows, not the literal text `wlan0`. Likewise, replace `<your-ssid>` with your actual network
name.

#### Test connectivity:
```shell
ping -c 3 archlinux.org
```

### 4.0 List Disks
```shell
lsblk
```
Lists the block devices (disks and existing partitions) the system can see, so you can identify
which one you're about to install onto. Look for your actual target disk, e.g. `/dev/nvme0n1`
for an NVMe drive or `/dev/sda` for a SATA/virtio one - `/dev/<your-disk>` in the rest of this
guide refers to whichever one you find here. Double-check you have the right disk: the next
step erases it.

### 5.0 Partition the Disk

**Want LVM instead?** This section through 7.0 Mount the Partitions (plus the swap and
initramfs-hooks steps later) sets up one plain ext4 root partition and a swapfile. If you'd
rather split root/var/tmp/swap/home into separate LVM volumes, skip ahead to
[appendices/lvm-disk-layout.md](appendices/lvm-disk-layout.md) instead of the steps below.

```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
`cfdisk` is an interactive partition editor. This creates the two partitions the core guide
needs: a small EFI System partition (for the bootloader) and one large Linux partition (for
everything else, formatted as ext4 in the next step).

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
After writing, `lsblk` again to see the two new partition device names. **They depend on your
disk type:** an NVMe disk like `/dev/nvme0n1` gets partitions named with a `p` before the
number (`/dev/nvme0n1p1`, `/dev/nvme0n1p2`), while a SATA/virtio disk like `/dev/sda` just gets
the number appended directly (`/dev/sda1`, `/dev/sda2`). The rest of this guide refers to these
as `/dev/<your-efi-partition>` and `/dev/<your-root-partition>` - substitute your actual
partition device names.

### 6.0 Format the Partitions
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/<your-root-partition>     # e.g. /dev/nvme0n1p2
```
Puts an actual filesystem on each partition: FAT32 on the EFI partition (required by the UEFI
spec for the boot partition) and ext4 on the root partition (this guide's filesystem of choice
for everything else).

### 7.0 Mount the Partitions
```shell
mount /dev/<your-root-partition> /mnt      # e.g. /dev/nvme0n1p2
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
Mounts the new filesystems at `/mnt` so `pacstrap` (next section) has somewhere to install the
new system into. `/boot` must remain unencrypted for UEFI boot, which is why it's a plain FAT32
partition rather than, say, part of an encrypted root.

## Base Installation

### Install Essential Packages

**Text editor choice:** the package list below includes a text editor, used in every
`nano ...` command throughout the rest of this guide. This guide defaults to `nano` because
it's simple and beginner-friendly; `neovim` and `vim` are common alternatives - if you'd rather
use one of those, swap `nano` for `neovim` or `vim` in the command below, and substitute your
editor of choice for `nano` in the editing commands used throughout the rest of this guide.

```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd openssh nano
```
`pacstrap` installs a minimal Arch package set into `/mnt`: the base system, the kernel and
firmware, the tool that builds your initramfs, shell completions, a DHCP client and the Wi-Fi
daemon (so networking works after reboot), SSH (optional - remove it unless you plan to use
it), and the text editor above.

## Configure the System

### 1.0 Generate fstab
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
`fstab` tells the system which filesystems to mount at boot and where. `genfstab` inspects what
you've already mounted under `/mnt` and writes the matching entries (keyed by UUID, via `-U`,
which is more reliable than device paths that can change) into the new system's `/etc/fstab`.

### 2.0 Chroot into New System
```shell
arch-chroot /mnt
```
Changes your working root into the new system at `/mnt`, so every command from here on runs
*inside* the system you're installing rather than the live ISO. All the remaining
"Configure the System" steps run inside this chroot.

### 3.0 Set Time and Locale
```shell
timedatectl set-ntp true
timedatectl set-timezone UTC # Avoids DST issues
hwclock --systohc --utc
```
Enables automatic clock sync (NTP), sets the system timezone, and writes the current time to
the hardware clock so it's correct across reboots. Pick the timezone that matches where you
actually are - run `timedatectl list-timezones` to see every option. A few regional examples:
`America/New_York`, `America/Toronto`, `Europe/London`. `UTC` (used above) is also a perfectly
valid choice if you'd rather sidestep daylight-saving time changes entirely.

#### Uncomment your locale(s) in /etc/locale.gen:
```shell
nano /etc/locale.gen
```
`/etc/locale.gen` lists every locale `locale-gen` (next step) is able to generate; only the
uncommented ones actually get built. This guide defaults to `en_US.UTF-8 UTF-8`. Common
alternatives: `en_GB.UTF-8 UTF-8` (British), `en_CA.UTF-8 UTF-8` (Canadian). Uncomment whichever
line(s) match the locale(s) you want.

#### Generate and set locale:
```shell
locale-gen
localectl set-locale LANG="en_US.UTF-8"
localectl set-locale LC_TIME="en_US.UTF-8"
echo "KEYMAP=us" > /etc/vconsole.conf
```
`locale-gen` builds the locale(s) you uncommented above; `localectl set-locale` makes one of
them the system default (`LANG` for general language/formatting, `LC_TIME` for date/time
formatting specifically). The `vconsole.conf` line sets the keymap for the text console on
every subsequent boot. Replace `en_US.UTF-8` with whichever locale you uncommented above, and
`us` with whichever keymap you chose back in step 2.0 of Pre-Installation (this keeps the
console keymap consistent between the live environment and the installed system).

### 4.0 Network Configuration
#### Set hostname:
```shell
echo <your-hostname> > /etc/hostname  # e.g. echo desktop > /etc/hostname
```
The hostname is the name your machine identifies itself by on a network. Replace
`<your-hostname>` with whatever you want to call this machine (e.g. `desktop`).

#### Edit /etc/hosts:
```shell
nano /etc/hosts
```
`/etc/hosts` is a local, static lookup table from hostnames to IP addresses, checked before DNS.
Add an entry mapping your own hostname to the loopback address so local tools that resolve
`<your-hostname>` still work without a network:

```shell
127.0.0.1   localhost
::1         localhost
127.0.1.1   <your-hostname>.localdomain   <your-hostname>  # e.g. desktop.localdomain desktop
```
Replace `<your-hostname>` with the same hostname you set above (in both places on the last
line).

### 5.0 Configure Swap (swapfile)
```shell
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```
A swapfile gives the kernel somewhere to page out memory under pressure, without needing a
dedicated swap partition/volume - the simplest approach when you're not using LVM. This creates
a 4G file (`count=4096` at `bs=1M`), a reasonable default for most systems; adjust the `count`
value to change the size (e.g. to roughly match your RAM if you want hibernation support).
`chmod 600` restricts it to root before `mkswap` formats it as swap space and `swapon` activates
it.

#### Verify:
```shell
swapon --show
```

#### Add it to fstab:
```shell
echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```
`genfstab` (step 1.0) ran before the swapfile existed, so it isn't in `/etc/fstab` yet; this
adds it manually so swap is activated automatically on every future boot.

### 6.0 Initramfs Configuration
#### Edit /etc/mkinitcpio.conf:
```shell
nano /etc/mkinitcpio.conf
```
The initramfs is a small, temporary root filesystem the kernel loads at boot to set up the
tools it needs before it can mount your real root filesystem. `mkinitcpio.conf` controls what
goes into it.

#### Place vfat into MODULES:
```conf
MODULES=(vfat)
```
Ensures FAT32 (used by the EFI partition) support is available early at boot.

#### Replace the systemd hook with udev and sd-vconsole with consolefont (ordering matters):
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```
The `HOOKS` array lists, in order, the stages the initramfs runs through to get your system
bootable - device discovery (`udev`), microcode loading, kernel modules, your keyboard/keymap
so you can type at a boot-time prompt if needed, and finally finding and checking your
filesystems. (No `lvm2` hook is needed here since this guide's core disk layout doesn't use
LVM - see the [LVM appendix](appendices/lvm-disk-layout.md) if you do.)

#### Rebuild initramfs:
```shell
mkinitcpio -P
```
Regenerates the initramfs for every installed kernel so the `HOOKS`/`MODULES` changes above
actually take effect.

### 7.0 Enable Networking Services
```shell
systemctl enable dhcpcd
systemctl enable iwd.service
systemctl enable sshd        # Enable if you installed openssh
```
These services were installed by `pacstrap` but aren't active yet; `enable` schedules them to
start automatically on every future boot, so you have networking (and optionally SSH access)
without manual intervention.

### 8.0 Set Root Password
```shell
passwd
```
Sets a password for the root account. You'll be prompted to type it (twice).

### 9.0 Add User

**Want a different shell instead of bash?** This guide defaults to bash (`-s /bin/bash` below)
since it's always present and needs no extra package. -> see
[appendices/alternate-shell.md](appendices/alternate-shell.md) for zsh/fish instead, then come
back and continue with 10.0 below.

```shell
useradd -m -G wheel -s /bin/bash <your-username>  # e.g. archie
passwd <your-username>                             # e.g. archie
```
Creates your everyday, non-root user account: `-m` creates a home directory, `-G wheel` adds
the user to the `wheel` group (which the next step grants elevated-privilege access to), and
`-s /bin/bash` sets bash as the login shell. Replace `<your-username>` with the username you
want, then set its password the same way you set root's.

### 10.0 Configure Privilege Escalation (sudo)
This guide uses `sudo` to let your user run commands as root - it's the most widely used
privilege-escalation tool, so this is a sensible default.

**Want doas instead?** `opendoas` is a smaller, simpler `sudo` alternative -> see
[appendices/alternate-privilege-escalation.md](appendices/alternate-privilege-escalation.md)
instead of the steps below.

```shell
pacman -S sudo
```

#### Allow the wheel group to run commands as root:
```shell
EDITOR=nano visudo
```
`visudo` opens `/etc/sudoers` for editing and validates its syntax before saving, which matters
because a broken `sudoers` file can lock you out of root access entirely - editing it with a
plain editor risks exactly that. Setting `EDITOR=nano` for this one command uses the editor you
installed back in Base Installation instead of `visudo`'s default `vi`; swap `nano` for your
editor of choice if you picked something else there.

In the editor, find and uncomment this line:
```conf
%wheel ALL=(ALL:ALL) ALL
```
This grants every member of the `wheel` group (which your user was added to in Add User)
permission to run any command as root via `sudo <command>`. Save and exit to write the change.

### 11.0 Install and Configure systemd-boot

This guide defaults to **systemd-boot** as its bootloader, since it's minimal and already part
of systemd. **Want Limine instead?** -> see
[appendices/limine-bootloader.md](appendices/limine-bootloader.md) instead of the steps below.

```shell
bootctl install
```
`bootctl install` copies systemd-boot's boot loader binary onto your EFI partition and
registers it with your system's UEFI firmware as the default boot entry.

#### Edit /boot/loader/loader.conf:
```shell
nano /boot/loader/loader.conf
```
`loader.conf` controls systemd-boot's own behavior (which entry boots by default, how long the
menu waits before auto-booting).

```conf
default arch.conf
timeout 3
console-mode max
editor no
```
`default` picks which boot entry (defined next) loads automatically; `timeout` is how many
seconds the boot menu waits before doing so; `console-mode max` uses the highest resolution text
mode available; `editor no` disables in-menu kernel command-line editing, a minor hardening
step so someone with physical access at boot can't alter boot parameters.

#### Create /boot/loader/entries/arch.conf:
```shell
nano /boot/loader/entries/arch.conf
```
A boot entry tells systemd-boot which kernel, initramfs, and kernel command line to use for a
given menu item.

```conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=/dev/<your-root-partition> rw
```
Replace `/dev/<your-root-partition>` with the actual root partition device you formatted back
in step 6.0 of Pre-Installation (e.g. `/dev/nvme0n1p2` or `/dev/sda2`) - not the literal text
`<your-root-partition>`.

#### Optional: Create a fallback entry, /boot/loader/entries/arch-fallback.conf:
```shell
nano /boot/loader/entries/arch-fallback.conf
```
A second entry pointing at the fallback initramfs (which includes a broader set of drivers/
modules), useful as a recovery option if a kernel update or configuration change breaks normal
boot.

```conf
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
options root=/dev/<your-root-partition> rw
```
Same substitution as above: replace `/dev/<your-root-partition>` with your actual root
partition device.

## Finalize and Reboot
#### Exit chroot:
```shell
exit
```
Leaves the chroot and returns you to the live ISO's shell.

#### Unmount all partitions:
```shell
umount -l /mnt
```
Unmounts everything under `/mnt` (`-l` lazily, so it succeeds even if something is still
briefly busy) before rebooting, so nothing is left half-written.

#### Reboot into the new system:
```shell
reboot
```
Remove the installation media when prompted so the system boots from disk into your new
install rather than back into the live ISO.

## Verify Installation
#### After logging in:
```shell
lsblk                          # Confirm partition layout
swapon --show                  # Verify swap active
bootctl status                 # Confirm systemd-boot is the active boot loader
cat /etc/fstab                 # Sanity-check mount entries
```
A quick sanity pass: your EFI and root partitions should be mounted as expected, the swapfile
should show as active, `bootctl status` should report systemd-boot as the current boot loader,
and `/etc/fstab` should list your root partition, EFI partition, and swapfile with no leftover
or unexpected entries.
