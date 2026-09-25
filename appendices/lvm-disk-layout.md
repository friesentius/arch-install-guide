# Appendix: LVM Disk Layout (root / var / tmp / swap / home)

This appendix is an alternative disk layout for the [core install guide](../arch-linux-install-guide.md).
Instead of a single ext4 root partition and a swapfile, it splits root, `/var`, `/tmp`, swap,
and `/home` into separate LVM logical volumes for better isolation.

**Use this appendix if:** you want separate volumes per top-level directory (e.g. so a runaway
log file in `/var` can't fill your root filesystem), or a separate swap volume instead of a
swapfile.

**Sections replaced:** this appendix replaces the core guide's sections 5.0-7.0 (Partition the
Disk / Format the Partitions / Mount the Partitions), the 5.0 Configure Swap section, and the
`HOOKS` line of the 6.0 Initramfs Configuration section. Everything else in the core guide -
keyboard layout, network setup, pacstrap, fstab, chroot, locale, hostname, networking services,
root password, user creation, privilege escalation, and the bootloader step - continues
unchanged around this appendix.

**Also needed:** add `lvm2` to the `pacstrap` package list in the core guide's Base Installation
step, so the installed system has the LVM tools available to assemble the volume group at boot.

## 1.0 Partition the Disk
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
|2      | 1130496        | 976773134    | 475.9G | 8309 | Linux Filesystem |
```

## 2.0 Create LVM Physical Volume & Volume Group
```shell
pvcreate /dev/nvme0n1p2
vgcreate vg /dev/nvme0n1p2
```

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
Adjust lvm volumes accordingly. (**Example:** "256G drive: reduce /var to 10G, /tmp to 4G")

## 4.0 Format Filesystems
```shell
mkfs.ext4 -L "Arch Root"   /dev/vg/root
mkfs.ext4 -L "Arch Var"    /dev/vg/var
mkfs.ext4 -L "Arch Tmp"    /dev/vg/tmp
mkfs.ext4 -L "Arch Home"   /dev/vg/home
mkswap /dev/vg/swap        # Format swap LV
```

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
mkfs.fat -F32 /dev/nvme0n1p1
mount /dev/nvme0n1p1 /mnt/boot
```
/boot must remain unencrypted for UEFI boot.

## 6.0 Enable Swap (after chroot, in place of the core guide's swapfile step)
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

## 7.0 Initramfs HOOKS
When you reach the core guide's Initramfs Configuration step, include the `lvm2` hook so the
initramfs can assemble the volume group before mounting root (ordering matters):
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block lvm2 filesystems fsck)
```

## 8.0 Secure /tmp Mount Options
```shell
nano /etc/fstab
```

#### Find /tmp and include noatime, nosuid, nodev:
```shell
UUID=example    /tmp    ext4    rw,relatime,noatime,nosuid,nodev    0 2
```
Prevents execution, device files, and suid abuse on /tmp.

## 9.0 (Optional) Clear /tmp on Boot
```shell
echo "D /tmp 1777 root root 1d" > /etc/tmpfiles.d/clean-tmp.conf
```
This uses systemd-tmpfiles to clean /tmp on boot. The 1d means files older than 1 day are
deleted. Change to 0 to clear all contents on every boot.

## Expected lsblk output
```shell
# output
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
