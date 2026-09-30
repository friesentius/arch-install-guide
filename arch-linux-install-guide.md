# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

This guide is one path from a live Arch ISO to a fully installed, configured system, written as a
sequence of copy-pasteable commands with plain-language explanations of what each step does and
why. It targets a UEFI system with a single disk.

It's laid out as **one path with branches**, not a core guide plus a pile of optional extras
tacked on at the end. Every high-level step below shows this guide's default choice inline; where
Arch offers a real alternative (a different disk layout, bootloader, shell, privilege-escalation
tool, or a post-install add-on like a firewall), that step calls it out with a **"Want X
instead?"** link to a branch file in [`branches/`](branches/README.md). Every branch ends with a
**"Continue in the main guide"** link back to the exact step you left, by number - take as many
scenic routes as you like, you always rejoin the same path. See
[`branches/README.md`](branches/README.md) for the full branch index, and the top-level
[`README.md`](README.md) for an overview of the whole repo.

**A note on placeholders:** anywhere you see a value wrapped in angle brackets, like
`/dev/<your-disk>` or `<your-username>`, it is a placeholder you must replace with the real
value for your system. Anything shown as a plain, unbracketed value (like `wlan0` or `vg`) is
just an example or a name this guide invents along the way; check the relevant command's
output (`lsblk`, `iwctl device list`, etc.) for your actual value before continuing.

## Pre-Installation

### 1.0 Set Keyboard Layout
```shell
loadkeys us
```
Sets the keyboard layout for the live installer session, so keys you type from here on
(including every command below and any passwords) map to the characters you expect. This is
the very first command in this guide for exactly that reason: it's the last thing worth typing
in the wrong layout. This guide defaults to a US keymap - if that's what you have, there's
nothing to do here, since it's already the live ISO's default; run the command anyway if you
want to be explicit. Common alternatives: `uk` (British), `ca` (Canadian French/English), `de`
(German), `dvorak` (US Dvorak), `dvorak-programmer` (Programmer Dvorak), `dvorak-l`/`dvorak-r`
(left-/right-handed one-handed Dvorak), `colemak`. Run `localectl list-keymaps` to see every
keymap available, then pass your chosen name to `loadkeys` in place of `us` above.

### 2.0 Verify UEFI Boot Mode
```shell
ls /sys/firmware/efi/efivars
```
This directory only exists if the system booted in UEFI mode rather than legacy BIOS. This
guide's later steps (the EFI partition, systemd-boot) only make sense on UEFI, so confirm this
first. If the directory doesn't exist, you're in BIOS/legacy mode and this guide doesn't apply
as written.

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

## Disk Layout

### 5.0 Choose Your Disk Layout
This guide's default is the simplest layout that works well for most single-disk systems: one
EFI partition, one ext4 root partition, and a swapfile (steps 6.0-8.0 below, then 14.0 Configure
Swap later). Two real alternatives exist, and the choice has to be made **now, before you
partition** - both change steps later in this guide, and retrofitting either after the fact means
backing up your data and starting over from this step.

**Want separate volumes for root/var/tmp/swap/home?** ->
[branches/lvm-disk-layout.md](branches/lvm-disk-layout.md) uses LVM instead of one root partition
and a swapfile, so a runaway log file in `/var` can't fill your root filesystem, and gives you a
separate swap volume instead of a swapfile. Replaces steps 6.0-8.0 and 14.0 below, and adds a
hook in 15.0.

**Want the disk encrypted?** -> [branches/disk-encryption.md](branches/disk-encryption.md) uses
LUKS so the disk's contents are unreadable without your passphrase if the machine is lost,
stolen, or physically accessed while powered off - the classic laptop case. Replaces steps
6.0-8.0 and 14.0, adds a hook in 15.0, and changes the boot entry in 20.0. It also covers
combining encryption with the LVM layout above (LVM-on-LUKS), if you want both.

Taking either branch? Read it in full before running any commands below - both explain exactly
which steps they replace and where you rejoin this guide. Sticking with the default? Continue to
6.0 below.

### 6.0 Partition the Disk
(Going with LVM or full-disk encryption instead? Each has its own partitioning step - use that
instead of this one.)

```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
`cfdisk` is an interactive partition editor. This creates the two partitions this guide's
default layout needs: a small EFI System partition (for the bootloader) and one large Linux
partition (for everything else, formatted as ext4 in the next step).

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

### 7.0 Format the Partitions
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/<your-root-partition>     # e.g. /dev/nvme0n1p2
```
Puts an actual filesystem on each partition: FAT32 on the EFI partition (required by the UEFI
spec for the boot partition) and ext4 on the root partition (this guide's filesystem of choice
for everything else).

### 8.0 Mount the Partitions
```shell
mount /dev/<your-root-partition> /mnt      # e.g. /dev/nvme0n1p2
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
Mounts the new filesystems at `/mnt` so `pacstrap` (next section) has somewhere to install the
new system into. `/boot` must remain unencrypted for UEFI boot, which is why it's a plain FAT32
partition rather than, say, part of an encrypted root.

## Base Installation

### 9.0 Install Essential Packages

**Text editor choice:** the package list below includes a text editor, used in every
`nano ...` command throughout the rest of this guide. This guide defaults to `nano` because
it's simple and beginner-friendly; `neovim` and `vim` are common alternatives - if you'd rather
use one of those, swap `nano` for `neovim` or `vim` in the command below, and substitute your
editor of choice for `nano` in the editing commands used throughout the rest of this guide.

Pick the command below that matches your disk layout - check now with `lsblk -f`: if you see
`LVM2_member` anywhere in the output (with or without `crypto_LUKS` above it), you're on LVM and
need the extra `lvm2` package below so the installed system can assemble the volume group at
boot; otherwise (plain `ext4`, or `crypto_LUKS` with no `LVM2_member`), use the plain list -
full-disk encryption alone needs nothing extra.

**Plain partition layout, or full-disk encryption without LVM:**
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd openssh nano
```

**LVM (with or without encryption):**
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd openssh nano lvm2
```

`pacstrap` installs a minimal Arch package set into `/mnt`: the base system, the kernel and
firmware, the tool that builds your initramfs, shell completions, a DHCP client and the Wi-Fi
daemon (so networking works after reboot), SSH (optional - remove it unless you plan to use
it), the text editor above, and (LVM only) the LVM tools.

## Configure the System

### 10.0 Generate fstab
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
`fstab` tells the system which filesystems to mount at boot and where. `genfstab` inspects what
you've already mounted under `/mnt` and writes the matching entries (keyed by UUID, via `-U`,
which is more reliable than device paths that can change) into the new system's `/etc/fstab`.

### 11.0 Chroot into New System
```shell
arch-chroot /mnt
```
Changes your working root into the new system at `/mnt`, so every command from here on runs
*inside* the system you're installing rather than the live ISO. All the remaining
"Configure the System" steps run inside this chroot.

### 12.0 Set Time and Locale
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
every subsequent boot. Replace `en_US.UTF-8` with whichever locale you uncommented above.

For `us`, use whatever keymap is currently active on your keyboard right now - test it directly:
type `@` (Shift+2 on a US layout) and `:` (Shift+; on a US layout); if both appear as expected,
use `us`; if either produces a different character, use that layout's name instead (the same
name you'd pass to `loadkeys`). This keeps the console keymap consistent between the live
environment and the installed system. This same `KEYMAP` value is also what the `keymap`
initramfs hook (see 15.0 Initramfs Configuration below) embeds for early-boot prompts - if
`lsblk -f` shows `crypto_LUKS` on your root partition (the full-disk-encryption branch), this is
what determines what you actually type at the LUKS passphrase prompt.

### 13.0 Network Configuration
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

Prefer NetworkManager over the `iwd`/`dhcpcd` setup below? That's a post-install swap, not a
decision you need to make now - see 29.0 System Configuration once you've finished the install.

### 14.0 Configure Swap
Pick the block below that matches your disk layout - check now with `lsblk -f`: `LVM2_member`
anywhere in the output means LVM, so use the swap-logical-volume block; anything else (plain
`ext4`, or `crypto_LUKS` alone) uses the swapfile block.

**Swapfile (plain partition layout, or full-disk encryption without LVM):**
```shell
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```
A swapfile gives the kernel somewhere to page out memory under pressure, without needing a
dedicated swap partition/volume. This creates a 4G file (`count=4096` at `bs=1M`), a reasonable
default for most systems; adjust the `count` value to change the size (e.g. to roughly match
your RAM if you want hibernation support). `chmod 600` restricts it to root before `mkswap`
formats it as swap space and `swapon` activates it. `genfstab` (10.0) ran before the swapfile
existed, so it isn't in `/etc/fstab` yet; the last line adds it manually so swap is activated
automatically on every future boot. If `lsblk -f` showed `crypto_LUKS` on your root partition,
this swapfile lives inside your already-encrypted root filesystem, so it's covered by the same
encryption automatically - no different commands needed.

**Swap logical volume (LVM, with or without encryption):**
```shell
swapon /dev/vg/swap
echo '/dev/vg/swap none swap defaults 0 0' >> /etc/fstab
```
Activates the `swap` logical volume you created while partitioning, and records it in
`/etc/fstab` so it's activated automatically on every future boot (a swap logical volume is a
block device `genfstab` usually picks up automatically, but adding it explicitly here
guarantees it).

#### Verify (either case):
```shell
swapon --show
```

### 15.0 Initramfs Configuration
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

#### Replace the systemd hook with udev and sd-vconsole with consolefont (ordering matters), and pick the line below that matches your disk layout - check now with `lsblk -f`: look for `crypto_LUKS` and/or `LVM2_member` in the output:

**Plain partition layout (default):**
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```

**LVM only:**
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block lvm2 filesystems fsck)
```

**Full-disk encryption only:**
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck)
```

**LVM + full-disk encryption (LVM-on-LUKS):**
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```

The `HOOKS` array lists, in order, the stages the initramfs runs through to get your system
bootable - device discovery (`udev`), microcode loading, kernel modules, your keyboard/keymap
so you can type at a boot-time prompt if needed, optionally unlocking an encrypted device and/or
assembling an LVM volume group, and finally finding and checking your filesystems. Ordering
matters: `encrypt` must come after `block` (which sets up the underlying block devices) and
`lvm2` must come after `encrypt` when both are present, since LVM needs the container unlocked
before it can find the volume group inside it - both come before `filesystems`. The `keymap`
hook specifically is what carries the `KEYMAP` you set in `/etc/vconsole.conf` (12.0 Set Time
and Locale, above) into the initramfs itself, so a non-US layout like Dvorak or Colemak still
applies at any prompt the initramfs shows before your real root filesystem is even mounted - the
case that matters in practice is typing a LUKS passphrase, if you're using the `encrypt` hook
above.

#### Rebuild initramfs:
```shell
mkinitcpio -P
```
Regenerates the initramfs for every installed kernel so the `HOOKS`/`MODULES` changes above
actually take effect.

### 16.0 Enable Networking Services
```shell
systemctl enable dhcpcd
systemctl enable iwd.service
systemctl enable sshd        # Enable if you installed openssh
```
These services were installed by `pacstrap` but aren't active yet; `enable` schedules them to
start automatically on every future boot, so you have networking (and optionally SSH access)
without manual intervention.

### 17.0 Set Root Password
```shell
passwd
```
Sets a password for the root account. You'll be prompted to type it (twice).

### 18.0 Add User

**Want a different shell instead of bash?** This guide defaults to bash (`-s /bin/bash` below)
since it's always present and needs no extra package. -> see
[branches/alternate-shell.md](branches/alternate-shell.md) for zsh/fish instead, then come
back and continue with 19.0 below.

```shell
useradd -m -G wheel -s /bin/bash <your-username>  # e.g. archie
passwd <your-username>                             # e.g. archie
```
Creates your everyday, non-root user account: `-m` creates a home directory, `-G wheel` adds
the user to the `wheel` group (which the next step grants elevated-privilege access to), and
`-s /bin/bash` sets bash as the login shell. Replace `<your-username>` with the username you
want, then set its password the same way you set root's.

### 19.0 Configure Privilege Escalation (sudo)
This guide uses `sudo` to let your user run commands as root - it's the most widely used
privilege-escalation tool, so this is a sensible default.

**Want doas instead?** `opendoas` is a smaller, simpler `sudo` alternative -> see
[branches/alternate-privilege-escalation.md](branches/alternate-privilege-escalation.md)
instead of the steps below, then rejoin at 20.0.

```shell
pacman -S sudo
```

#### Allow the wheel group to run commands as root:
```shell
EDITOR=nano visudo
```
`visudo` opens `/etc/sudoers` for editing and validates its syntax before saving, which matters
because a broken `sudoers` file can lock you out of root access entirely - editing it with a
plain editor risks exactly that. Setting `EDITOR=nano` for this one command uses `nano` instead
of `visudo`'s default `vi`; check which editor you actually have with
`pacman -Q nano neovim vim 2>/dev/null`, and swap `nano` above for `neovim` or `vim` if that's
what the check shows instead.

In the editor, find and uncomment this line:
```conf
%wheel ALL=(ALL:ALL) ALL
```
This grants every member of the `wheel` group (which your user was added to in 18.0 Add User)
permission to run any command as root via `sudo <command>`. Save and exit to write the change.

## Boot Loader

### 20.0 Install and Configure systemd-boot

This guide defaults to **systemd-boot** as its bootloader, since it's minimal and already part
of systemd. **Want Limine instead?** -> see
[branches/limine-bootloader.md](branches/limine-bootloader.md) instead of the steps below, then
rejoin at 21.0.

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

#### Find your options line
First, work out the `options` line your boot entry needs. Check your disk layout now with
`lsblk -f`: look for `crypto_LUKS` and/or `LVM2_member` in the output. If you see `crypto_LUKS`
(with or without `LVM2_member` alongside it), first find your root partition's UUID (the
underlying encrypted partition's UUID, not the mapper device's):
```shell
blkid /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```

**Plain partition layout (default):**
```conf
options root=/dev/<your-root-partition> rw
```
Replace `/dev/<your-root-partition>` with the actual root partition device you formatted back
in 7.0 Format the Partitions (e.g. `/dev/nvme0n1p2` or `/dev/sda2`).

**LVM only:**
```conf
options root=/dev/vg/root rw
```

**Full-disk encryption only:**
```conf
options cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/mapper/cryptroot rw
```
Replace `<your-root-partition-uuid>` with the UUID `blkid` printed above.
`cryptdevice=UUID=...:cryptroot` tells the initramfs's `encrypt` hook which device to unlock (by
UUID, since raw device names can shift) and what to name the resulting mapper device
(`cryptroot`, matching what you named it when you ran `cryptsetup open` while partitioning);
`root=/dev/mapper/cryptroot` then points the kernel at the now-unlocked device.

**LVM + full-disk encryption (LVM-on-LUKS):**
```conf
options cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/vg/root rw
```
Same `cryptdevice=` as above (replace `<your-root-partition-uuid>` with your `blkid` output),
but `root=` points at the LVM logical volume, which the initramfs's `lvm2` hook finds inside the
now-unlocked container.

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
Replace the `options` line with whichever one you worked out above for your disk layout.

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
Same as above: replace the `options` line with the same one you used in `arch.conf`.

## Finalize and Reboot

### 21.0 Exit Chroot
```shell
exit
```
Leaves the chroot and returns you to the live ISO's shell.

### 22.0 Unmount All Partitions
```shell
umount -l /mnt
```
Unmounts everything under `/mnt` (`-l` lazily, so it succeeds even if something is still
briefly busy) before rebooting, so nothing is left half-written.

### 23.0 Reboot into the New System
```shell
reboot
```
Remove the installation media when prompted so the system boots from disk into your new
install rather than back into the live ISO.

## Verify Installation

### 24.0 Verify Installation
#### After logging in:
```shell
lsblk                          # Confirm partition layout
swapon --show                  # Verify swap active
cat /etc/fstab                 # Sanity-check mount entries
```
Then check which bootloader you're running:
```shell
ls /boot/limine.conf 2>/dev/null && echo "Limine" || bootctl status
```
If `/boot/limine.conf` exists, you're on Limine - it doesn't register itself with systemd, so
there's no status command to run; simply having reached this login prompt confirms it worked.
Otherwise, `bootctl status` should report systemd-boot as the active boot loader.

A quick sanity pass: your EFI and root partitions should be mounted as expected, `swapon --show`
should show either a swapfile or a swap logical volume as active (whichever you set up), and
`/etc/fstab` should list your root partition, EFI partition, and swap with no leftover or
unexpected entries.

## Post-Install Configuration

Your base system is installed, bootable, and verified. Everything below is still part of one
path, just an optional one: each step defaults to "skip it, your system already works," with a
branch if you want more. Walk through them in order, or jump straight to the one you want - they
don't depend on each other except where noted. See
[`branches/README.md`](branches/README.md) for the full index.

### 25.0 Graphics, Microcode, and AUR Tools
Default: skip - a minimal Arch install boots and runs fine without CPU microcode updates, a GPU
driver package, or AUR build tools. Most desktop and gaming setups want at least one of these,
though.

**Want them?** -> [branches/graphics-and-extras.md](branches/graphics-and-extras.md) covers CPU
microcode, graphics drivers (with 32-bit/multilib support for things like Steam), and the AUR
build toolchain. Can also be done from inside the chroot before 21.0 Exit Chroot, if you'd
rather do it before first boot.

Continue to 26.0 below either way.

### 26.0 Firewall
Default: skip - nothing here is required to use the system.

**Want one?** -> [branches/firewall.md](branches/firewall.md) sets up `ufw` with a
default-deny-inbound posture: nothing gets in unless you explicitly allow it.

Continue to 27.0 below either way.

### 27.0 SSH Hardening
Check now whether this applies to you: `systemctl is-enabled sshd 2>/dev/null`. If it doesn't
print `enabled`, you didn't set up SSH and nothing here applies. Default: skip.

**Want to lock it down?** -> [branches/ssh-hardening.md](branches/ssh-hardening.md) covers
key-based login, disabling password/root login, and `fail2ban` against brute-force attempts.

Continue to 28.0 below either way.

### 28.0 Update Hygiene
Default: skip, and just run `pacman -Syu` periodically, prefixed with whichever
privilege-escalation command is actually installed - check with `which sudo doas 2>/dev/null` -
and reading [archlinux.org/news](https://archlinux.org/news/) first. Arch is a rolling release,
so it needs *some* attention, but fully unattended upgrades are a real risk here (see the branch
below for why).

**Want a scheduled reminder instead of relying on memory?** ->
[branches/automatic-updates.md](branches/automatic-updates.md) sets up a timer that checks for
updates (without applying them) and covers reading the news before you actually upgrade.

Continue to 29.0 below either way.

### 29.0 System Configuration
Default: keep this guide's `iwd`/`dhcpcd` networking, `systemd-timesyncd` time sync, and no
power-management daemon.

**Want alternatives?** -> [branches/system-config.md](branches/system-config.md) covers
NetworkManager (better for a desktop environment's network widget), power management
(`power-profiles-daemon` or TLP, mainly for laptops), and `chrony` (more configurable time
sync). Pick any, none, or all three independently.

Continue to 30.0 below either way.

### 30.0 Shell Configuration and Dotfiles
Default: skip - your shell works fine with its stock config.

**Want to customize it?** -> [branches/shell-config.md](branches/shell-config.md) covers your
shell's config file (`.bashrc`/`.zshrc`/`config.fish`) and managing dotfiles long-term - check
`echo $SHELL` now to see which section applies to you. Pairs with
[branches/alternate-shell.md](branches/alternate-shell.md) if you want to switch shells first.

This is the last step on the path.

## You're Done

However you got here - straight down the default path, or by way of LVM, encryption, Limine,
doas, a different shell, and any of the post-install branches - you now have a bootable Arch
system configured the way you chose.

**Wanted full-disk encryption but didn't take that branch?** It's a pre-install decision - see
5.0 Choose Your Disk Layout. Adding it now means backing up your data, repartitioning, and
reinstalling from there; there's no in-place way to encrypt a live root filesystem.
