# Branch: LVM on LUKS (Encrypted LVM Volumes)

Forks from the [main guide](../arch-linux-install-guide.md)'s 5.0 Choose Your Disk Layout. It
encrypts one partition with LUKS, then splits the unlocked container into separate LVM volumes
for root, `/var`, `/tmp`, swap, and `/home`. One passphrase unlocks everything, and a runaway
`/var` can't fill root. The EFI partition stays unencrypted, because UEFI firmware must read it.
The volume group and volume names (`vg`, `root`, `var`, `tmp`, `swap`, `home`) are names you
create here.

## 6.0 Partition the Disk
```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
Create a small EFI System partition for the bootloader and one Linux partition to hold the
encrypted container:

```shell
# delete existing partition(s) to make room for your new partition scheme
select [ Delete ]

# Set up the EFI system partition
select [ New ]

Partition Size: 1G

select [ Type ] "EFI System"

# Set up the partition that will become the LUKS container
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
(`/dev/nvme0n1p1`); SATA/virtio disks don't (`/dev/sda1`). The rest of this branch calls them
`/dev/<your-efi-partition>` and `/dev/<your-root-partition>`.

#### Create the LUKS container:
```shell
cryptsetup luksFormat /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```
Type `YES` to confirm, then set a passphrase. **There is no recovery if you forget it**, so store
it somewhere safe, such as a password manager.

#### Open the LUKS container:
```shell
cryptsetup open /dev/<your-root-partition> cryptroot  # e.g. /dev/nvme0n1p2
```
Unlocks the container as `/dev/mapper/cryptroot`. `cryptroot` is a name you choose here; the
boot entry in 15.0 uses the same name.

#### Create the LVM physical volume and volume group:
```shell
pvcreate /dev/mapper/cryptroot
vgcreate vg /dev/mapper/cryptroot
```
Marks it as LVM storage and groups it into a volume group named `vg`.

#### Create logical volumes:
```shell
# Root: OS and packages (20G)
lvcreate -L 20G vg -n root

# /var: logs, caches, databases (20G)
lvcreate -L 20G vg -n var

# /tmp: temporary files (8G)
lvcreate -L 8G vg -n tmp

# Swap: 4G (adjust to match RAM if hibernating)
lvcreate -L 4G vg -n swap

# /home: remaining space
lvcreate -l 100%FREE vg -n home
```
Logical volumes work like partitions but can be resized later. Adjust the sizes to your drive;
on a 256G drive, for example, 10G `/var` and 4G `/tmp` leave more for `/home`.

## 7.0 Format the Partitions
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 -L "Arch Root"   /dev/vg/root
mkfs.ext4 -L "Arch Var"    /dev/vg/var
mkfs.ext4 -L "Arch Tmp"    /dev/vg/tmp
mkfs.ext4 -L "Arch Home"   /dev/vg/home
mkswap /dev/vg/swap
```
UEFI requires FAT32 on the EFI partition; each volume gets ext4, and the swap volume is formatted
as swap.

## 8.0 Mount the Partitions
```shell
# Mount root first:
mount /dev/vg/root /mnt

# Create and mount other directories:
mkdir -p /mnt/{home,var,tmp,boot}
mount /dev/vg/home /mnt/home
mount /dev/vg/var  /mnt/var
mount /dev/vg/tmp  /mnt/tmp
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
The new system gets installed under `/mnt` in the next step. `lsblk` should now look like this:
```shell
# example output - your disk name and sizes will differ
NAME             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
nvme0n1          259:0    0 476.9G  0 disk
├─nvme0n1p1      259:1    0     1G  0 part  /boot
└─nvme0n1p2      259:2    0 475.9G  0 part
  └─cryptroot    254:0    0 475.9G  0 crypt
    ├─vg-root    254:1    0    20G  0 lvm   /
    ├─vg-var     254:2    0    20G  0 lvm   /var
    ├─vg-tmp     254:3    0     8G  0 lvm   /tmp
    ├─vg-swap    254:4    0     4G  0 lvm
    └─vg-home    254:5    0 423.9G  0 lvm   /home
```

## 9.0 Install Essential Packages
```shell
pacstrap /mnt base linux linux-firmware mkinitcpio bash-completion dhcpcd iwd nano lvm2
```
Installs the base system, kernel, firmware, initramfs builder, shell completions, networking
(`dhcpcd`, `iwd`), the `nano` text editor, which this guide's commands use, and `lvm2`, which
the installed system needs to find its volumes at boot. Add `neovim` or `vim` to the list if you
want one of them as well.

## 10.0 Generate fstab
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
Writes the mounts under `/mnt` into the new system's `/etc/fstab`, keyed by UUID (`-U`) since
device names can change between boots.

#### Optional: harden /tmp:
`/tmp` has its own volume here, so it can get stricter mount options. Open the new fstab:
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

## 11.0 Chroot into New System
```shell
arch-chroot /mnt
```
Every command from here until 21.0 runs inside the new system instead of the live ISO.

## 12.0 Set Time, Locale, and Keymap
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
console and at the disk passphrase prompt at every boot: `us` for a US keyboard, otherwise e.g.
`uk`, `ca`, `de`, `dvorak`, `dvorak-programmer`, `dvorak-l`/`dvorak-r`, or `colemak`.

## 13.0 Configure Swap
```shell
swapon /dev/vg/swap
echo '/dev/vg/swap none swap defaults 0 0' >> /etc/fstab
```
Activates the `swap` logical volume and adds it to fstab.

#### Verify:
```shell
swapon --show
```

## 14.0 Initramfs Configuration
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
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```
Hooks run in order. `encrypt` must come after `block`, and `lvm2` after `encrypt`, since the
volume group is only visible once the container is unlocked; both come before `filesystems`. This
line uses `udev`, `keymap`, and `consolefont` in place of the stock `systemd` and `sd-vconsole`
hooks; `keymap` copies the `KEYMAP` from `/etc/vconsole.conf` into the initramfs, so your layout
applies when you type the passphrase. `microcode` embeds CPU microcode updates, if installed.

#### Rebuild initramfs:
```shell
mkinitcpio -P
```

## 15.0 Install and Configure systemd-boot

**Want Limine instead?** -> [limine-bootloader-lvm-on-luks.md](limine-bootloader-lvm-on-luks.md)

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

#### Find your root partition's UUID:
```shell
blkid /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```
Use the UUID of the encrypted partition itself, not of `/dev/mapper/cryptroot`.

#### Create /boot/loader/entries/arch.conf:
```shell
nano /boot/loader/entries/arch.conf
```
```conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/vg/root rw
```
`cryptdevice=` tells the `encrypt` hook which partition to unlock (replace
`<your-root-partition-uuid>` with the UUID `blkid` printed) and to name it `cryptroot`; `root=`
then points at the root logical volume inside it.

#### Optional: Create a fallback entry, /boot/loader/entries/arch-fallback.conf:
```shell
nano /boot/loader/entries/arch-fallback.conf
```
```conf
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
options cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/vg/root rw
```
Boots the fallback initramfs, which includes more drivers - a recovery option if an update breaks
normal boot. Use the same `options` line as `arch.conf`.

## Continue in the main guide
Continue at [16.0 Network Configuration](../arch-linux-install-guide.md#160-network-configuration).
