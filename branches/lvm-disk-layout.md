# Branch: LVM Disk Layout (root / var / tmp / swap / home)

Forks from the [main guide](../arch-linux-install-guide.md)'s 5.0 Choose Your Disk Layout and
replaces steps 6.0-8.0. Instead of one ext4 root partition and a swapfile, it splits root,
`/var`, `/tmp`, swap, and `/home` into separate LVM logical volumes, so a runaway `/var` can't
fill root and swap gets its own volume.

`/dev/<your-disk>` and its partitions are placeholders for your device names (find them with
`lsblk`). The volume group and volume names (`vg`, `root`, `var`, `tmp`, `swap`, `home`) are
names you create here, and later steps assume them.

Want encryption too? Start with [disk-encryption.md](disk-encryption.md) instead; it sends you
here at the right point.

## 1.0 Partition the Disk
```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
Create an EFI System partition and one Linux LVM partition:

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

select [ Type ] "Linux LVM"

select [ Write ]
# example cfdisk output - your sizes and disk name will differ
|Number | Start (sector) | End (sector) | Size   | Code | Name             |
|------ | -------------- | ------------ | ------ | ---- | ---------------- |
|1      | 2048           | 1130495      | 1G     | EF00 | EFI System       |
|2      | 1130496        | 976773134    | 475.9G | 8E00 | Linux LVM        |
```
Run `lsblk` for the new partition names (`/dev/<your-efi-partition>` and
`/dev/<your-lvm-partition>` below). NVMe disks add a `p` before the number; SATA/virtio disks
don't.

## 2.0 Create LVM Physical Volume & Volume Group
```shell
pvcreate /dev/<your-lvm-partition>       # e.g. /dev/nvme0n1p2
vgcreate vg /dev/<your-lvm-partition>    # e.g. /dev/nvme0n1p2
```
Marks the partition as LVM storage and groups it into a volume group named `vg`.

## 3.0 Create Logical Volumes
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
mkfs.fat -F32 /dev/<your-efi-partition>       # e.g. /dev/nvme0n1p1
mount /dev/<your-efi-partition> /mnt/boot     # e.g. /dev/nvme0n1p1
```

#### Expected lsblk output:
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
Continue at [9.0 Install Essential Packages](../arch-linux-install-guide.md#90-install-essential-packages).
Steps 9.0, 10.0, 14.0, 15.0, and 20.0 each list a variant per disk layout, with an `lsblk -f`
check that tells you which one is yours.
