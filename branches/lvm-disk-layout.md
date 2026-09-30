# Branch: LVM Disk Layout (root / var / tmp / swap / home)

This branch forks from the [main guide](../arch-linux-install-guide.md)'s 5.0 Choose Your Disk
Layout step. Instead of a single ext4 root partition and a swapfile, it splits root, `/var`,
`/tmp`, swap, and `/home` into separate LVM logical volumes for better isolation.

**Use this branch if:** you want separate volumes per top-level directory (e.g. so a runaway
log file in `/var` can't fill your root filesystem), or a separate swap volume instead of a
swapfile.

**Steps replaced:** this branch replaces the main guide's 6.0-8.0 (Partition the Disk / Format
the Partitions / Mount the Partitions), its 14.0 Configure Swap step, and the `HOOKS` line of
its 15.0 Initramfs Configuration step. Everything else in the main guide - keyboard layout,
network setup, pacstrap, fstab, chroot, locale, hostname, networking services, root password,
user creation, privilege escalation, and the bootloader step - continues unchanged around this
branch.

**Also needed:** add `lvm2` to the `pacstrap` package list in the main guide's 9.0 Install
Essential Packages step, so the installed system has the LVM tools available to assemble the
volume group at boot.

**Placeholders:** as in the main guide, `/dev/<your-disk>` and its derived partitions are
placeholders for your actual device names - see the main guide's List Disks / Partition the
Disk steps for how to find them. This branch's logical volume names (`vg`, `root`, `var`,
`tmp`, `swap`, `home`) are names *you* create in the steps below, not values to look up, so
they're shown as plain text; you can rename them if you like, but the rest of this branch
assumes the names shown here.

## 1.0 Partition the Disk
```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
Same idea as the main guide: an EFI System partition, plus one Linux partition - except here
the second partition becomes an LVM physical volume instead of being formatted directly.

```shell
# delete existing partition(s) to make room for your new partition scheme
select [ Delete ]

# Set up the EFI system partition
select [ New ]

Partition Size: 1G

select [ Type ] "EFI System"

# Set up the LVM partition
select [ New ]

Partition Size: accept default value (uses the remaining free space)

select [ Write ]
# example cfdisk output - your sizes and disk name will differ
|Number | Start (sector) | End (sector) | Size   | Code | Name             |
|------ | -------------- | ------------ | ------ | ---- | ---------------- |
|1      | 2048           | 1130495      | 1G     | EF00 | EFI System       |
|2      | 1130496        | 976773134    | 475.9G | 8309 | Linux Filesystem |
```
As in the main guide, run `lsblk` after writing to find your actual partition device names
(`/dev/<your-efi-partition>` and `/dev/<your-lvm-partition>` below) - NVMe disks get a `p`
before the partition number, SATA/virtio disks don't.

## 2.0 Create LVM Physical Volume & Volume Group
```shell
pvcreate /dev/<your-lvm-partition>       # e.g. /dev/nvme0n1p2
vgcreate vg /dev/<your-lvm-partition>    # e.g. /dev/nvme0n1p2
```
`pvcreate` marks the partition as an LVM physical volume (LVM's raw storage unit); `vgcreate`
groups it into a volume group named `vg`, from which the logical volumes below are carved out.

## 3.0 Create Logical Volumes
#### Create dedicated logical volumes for better isolation and security:
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
Each `lvcreate` carves a logical volume - LVM's equivalent of a partition, but resizable later
without repartitioning - out of the `vg` volume group. The sizes above are reasonable defaults
for a mid-size drive; adjust them to fit yours. (**Example:** on a 256G drive, you might reduce
`/var` to 10G and `/tmp` to 4G to leave more room for `/home`.)

## 4.0 Format Filesystems
```shell
mkfs.ext4 -L "Arch Root"   /dev/vg/root
mkfs.ext4 -L "Arch Var"    /dev/vg/var
mkfs.ext4 -L "Arch Tmp"    /dev/vg/tmp
mkfs.ext4 -L "Arch Home"   /dev/vg/home
mkswap /dev/vg/swap        # Format swap LV
```
Formats each logical volume as ext4 (matching the main guide's filesystem choice) and the swap
volume as swap space. `/dev/vg/<name>` addresses a logical volume by the volume group and
volume names you chose in the previous step.

## 5.0 Mount Filesystems
```shell
# Mount root first:
mount /dev/vg/root /mnt

# Create and mount other directories:
mkdir -p /mnt/{home,var,tmp,boot}
mount /dev/vg/home /mnt/home
mount /dev/vg/var  /mnt/var
mount /dev/vg/tmp  /mnt/tmp

# Format and mount EFI partition:
mkfs.fat -F32 /dev/<your-efi-partition>       # e.g. /dev/nvme0n1p1
mount /dev/<your-efi-partition> /mnt/boot     # e.g. /dev/nvme0n1p1
```
Mounts each volume at the directory it corresponds to, so `pacstrap` installs onto the full
layout. `/boot` must remain unencrypted for UEFI boot, same as in the main guide.

**Continue in the main guide:** your disk layout is done. Jump to
[9.0 Install Essential Packages](../arch-linux-install-guide.md#90-install-essential-packages)
and follow the main guide normally through fstab, chroot, time/locale, and hostname setup. Come
back here when you reach
[14.0 Configure Swap](../arch-linux-install-guide.md#140-configure-swap-swapfile) - use 6.0
below instead of that step.

## 6.0 Enable Swap (after chroot, in place of the main guide's swapfile step)
#### Activate:
```shell
swapon /dev/vg/swap
```

#### Verify:
```shell
swapon --show
```

#### Ensure it's in fstab:
```shell
echo '/dev/vg/swap none swap defaults 0 0' >> /etc/fstab
```
Unlike a swapfile, a swap logical volume is a block device `genfstab` picks up automatically in
most cases - but adding it explicitly here guarantees it, same rationale as the main guide's
swapfile fstab entry.

## 7.0 Initramfs HOOKS
When you reach the main guide's 15.0 Initramfs Configuration step, include the `lvm2` hook so the
initramfs can assemble the volume group before mounting root (ordering matters - it must come
before `filesystems`, after `block`):
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block lvm2 filesystems fsck)
```

## 8.0 Secure /tmp Mount Options
```shell
nano /etc/fstab
```
Since `/tmp` is its own volume here (rather than part of root), it's worth locking down its
mount options.

#### Find the /tmp line and add noatime, nosuid, nodev:
```shell
UUID=<your-tmp-volume-uuid>    /tmp    ext4    rw,relatime,noatime,nosuid,nodev    0 2    # e.g. UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890
```
`<your-tmp-volume-uuid>` is whatever UUID `genfstab` already wrote for `/dev/vg/tmp` in this
line - don't replace the whole line, just add `noatime,nosuid,nodev` to its options. This
prevents executing binaries, creating device files, and honoring setuid/setgid bits on `/tmp`,
which is meaningful hardening for a world-writable directory.

## 9.0 (Optional) Clear /tmp on Boot
```shell
echo "D /tmp 1777 root root 1d" > /etc/tmpfiles.d/clean-tmp.conf
```
This uses systemd-tmpfiles to clean `/tmp` on boot. The `1d` means files older than 1 day are
deleted; change it to `0` to clear all contents on every boot.

## Expected lsblk output
```shell
# example output - your disk name and sizes will differ
NAME          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
nvme0n1       259:0    0 476.9G  0 disk
├─nvme0n1p1   259:1    0     1G  0 part  /boot
└─nvme0n1p2   259:2    0 475.9G  0 part
  └─vg-root   254:1    0    20G  0 lvm   /
  └─vg-var    254:2    0    20G  0 lvm   /var
  └─vg-tmp    254:3    0     8G  0 lvm   /tmp
  └─vg-swap   254:4    0     4G  0 lvm
  └─vg-home   254:5    0 423.9G  0 lvm   /home
```

## Continue in the main guide
You've now covered the main guide's disk layout, swap, and initramfs `HOOKS` steps with their
LVM equivalents. Skip the main guide's own 14.0 Configure Swap and the `HOOKS` line of 15.0
Initramfs Configuration (you've just done both above), and pick back up at
[16.0 Enable Networking Services](../arch-linux-install-guide.md#160-enable-networking-services).
